# 服务器设计总结：

线程创建与线程处理函数：

```c++
#include<pthread.h>
int pthread_create(pthread_t *thread_tid, //返回生成线程的id
               const pthread_attr_t *attr,//指向线程属性的指针，通常为NULL
               void *(*start_routine)(void*) //线程处理函数的地址
              void *arg) //start_routine函数中的参数

//函数原型中的第三个参数，为函数指针，指向处理线程函数的地址。该函数，要求为静态函数。如果处理线程函数为类成员函数时，需要将其设置为静态成员函数。
```

线程池设计：

线程池为半同步/半反应堆，反应堆具体模式为proactor事件处理模式。

主线程充当异步线程，负责监听文件描述符，接受socket新连接，并向内核事件表注册事件，若监听的socket发生了读写事件，将任务（socket描述符）插入请求队列，工作线程从请求队列中取出任务，完成数据的处理。【初始化请求连接类对象，包括max_connection个文件描述符】

Linux定时方法：

（1）socket选项SO_RECVTIMO 和 SO_SNDTIMO

（2）SIGALRM信号：alarm函数周期性地触发SIGALRM信号，信号处理函数利用管道通知主循环，主循环接收到该信号后对升序链表上作用定时器进行处理，若该段时间内没有数据较好，则关闭该连接，释放占用的资源。

（3）IO复用系统调用的超时参数

**说明socket网络编程有哪些系统调用？其中close是一次就能直接关闭的吗，半关闭状态是怎么产生的？**

由于是双向的，两边都要发FIN，服务器关闭socket，用close会将该socket的计数-1，如果引用还是大于0，那么socket端口状态保持不变，如果为0，会将sender缓冲中的数发出去，然后发送FIN。可能在多进程中出现半关闭，所以应该使用

shutdown(sockfd, SHUT_RDWR); close(sockfd);

shutdown不考虑描述符的引用计数，直接关闭描述符，到**LAST_ACK状态**。也可选择中止一个方向的连接，只中止读或只中止写。 

如果有**多个进程**共享一个套接字，close每被调用一次，计数减1，直到计数为0时，也就是所用进程都调用了close，套接字将被释放。

在多进程中如果一个进程调用了shutdown(sfd, SHUT_RDWR)后，其它的进程将无法进行通信。但，如果一个进程close(sfd)将不会影响到其它进程。

就是说可能会有多个进程共享使用一个socket。其它的系统调用有

```
socket(AF_INET, SOCK_STREAM, 0) AF表示地址族，初始化套接口地址结构； socket (int namespace, int style, int protocol)
```

客户端：socket，connect，send，recv，close

服务器：多了bind，listen，accept

客户端服务器socket系统调用示意图：

<img src="D:\C++游戏编程\tcpapi.png" alt="tcpapi" style="zoom: 50%;" />



## 线程池使用方式：

理解以下几点：

（1）listen()系统调用后将客户端连接放入backlog连接队列中，队列大小一般为2-20，里面包含syn_send状态和estiblish状态。返回值为监听的文件描述符。只有经过accept()系统调用之后返回新的connfd，才是真正三次握手建立连接，connfd才是最终服务器要处理IO操作的connfd.

（2）定义了一个http_server的类，其成员变量包括m_epollfd，m_sockfd，用于表示该连接类具体的内核事件表和文件描述符。

（3）实际操作时首先new一个 http_server * users=new http_server[MAX_FD]的指向并发连接的数组的指针，预先为每一个连接分配一个http_server对象；所有的对象都放在这个users数组里面。然后，每个对象可以调用自身内的init()注册事件，process()等方法，然后线程池也是如此，从users里面取出http_server连接对象之后，调用处理函数，完成IO处理和请求解析以及请求返回。

（4）定时器使用绝对时间表示超时时间（即，连接时刻+TIMESLOT，可理解为每个连接的存活时间）采用升序链表；

（5）统一事件：异常事件，IO事件，信号事件到epollfd内核事件表，一次只能处理其中一种。

（6）定时事件处理时将此刻事件与链表头至链表尾部比较大小，删除存活时间小于当前时间的连接资源。

（7）若有IO发生，将该连接的定时器向后移动3*TIMESLO时间。即增加存活时间。

定时器使用方式：