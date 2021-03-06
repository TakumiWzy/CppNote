# Linux操作系统总结：

1.Linux内核组成？

一个完整的Linux内核通常由5部分组成：内存管理，进程管理，进程间通信，虚拟文件系统和网络接口。

2.什么是内核态，用户态？
https://www.cnblogs.com/maxigang/p/9041080.html
内核态：CPU可以访问内存中的所有数据，包括外围设备，如：硬盘，网卡，CPU也可以将自己从一个程序切换到另外一个程序。（当CPU执行内核的代码）
用户态：只能受限地访问内存，且不允许访问外围设备，占用CPU的能力被剥夺，CPU资源可以被其他程序夺取。（当CPU执行的是用户写的代码）
3.用户和内核通信方式？
Linux用户态和内核态由于CPU权限的限制，通信不像使用进程间通信的方式那么简单。通常用户代码在用户空间，通过系统调用函数来访问内核空间，这是最常用的一种用户和内核态通信的方式。除此之外，还有四种方式：
 + procfs(/proc)
 + sysctl(/proc/sys)
 + sysfs(/sys)
 + netlink 套接口

4.系统调用作用？read()/write()内核做了哪些事情？
用于用户和内核间通信。
5.中断类别？

6.Linux下系统调用与中断关系？

7.为什么要Bootloader？

8.Linux内核同步方式？

9.大小端区别？
大端序：网络序，先存放高地址，后存放低地址。
小端序：主机序，先存放低地址，后存放高地址。
10.为什么自旋锁不能睡眠，而在拥有信号量时就可以？

11.Linux检查内存状态？

12.程序从开始运行到结束完整四个过程？

13.死锁原因，条件，如何预防？

14.硬链接与软链接？

15.虚拟地址和物理地址区别与联系？

16.计算机32bit和64bit区别？

内存寻址范围不同。

17.中断和异常区别？

中断是指CPU对系统发生某件事的响应。

中断又分为外中断和内中断，外中断：外部设备事件引起如IO，控制台终端中断，打印机中断。（即不是由程序指令引起中断）。内中断：即异常，由CPU内部事件引起的中断，如程序出错，非法指令，地址越界，内中断也被译为“捕获”或“陷入”。

18.Linux操作系统挂起，休眠，关机命令？

19.如何查看文件前n行？如何查看文件后n行？

head -n xxx.txt

tail -n xxx.txt

20.shell脚本如何批量创建文件，如何批量创建目录？

touch xxx{1..100}

mkdir xxx{1..100}

21.进程通信方式？

22.匿名管道和命名管道区别？如何创建管道？

int pipefd[2];

pipe(pipefd);

mkfifo	xxx

23.Linux三种标准输入输出？

STDIN STDOUT STDERR

文件描述符是多少0,1,2

24.Linux开机启动过程？

25.Linux硬盘格式化过程？

26.IO复用方式有哪几种？区别？

select，poll，epoll。

select，FD_SET数组描述文件描述符采用bitmap数组每一位对应一个文件描述符状态（0，1），上限受到数组大小限制，若要修改大小，需要重写编译内核。需要遍历查找就绪文件描述符，只支持水平触发模式。

poll，采用链表描述文件描述符，无大小限制，使用遍历方式查找就绪文件描述符，只支持水平触发模式。

epoll，一组函数：epoll_create()，epoll_ctl，epoll_wait()，使用红黑树加一条链表(events)描述文件描述符，采用epoll_wait()函数调用直接返回就绪文件描述符，无大小限制。支持水平触发和边缘触发模式。

```c++
#include<sys/epoll.h>
int epoll_create(int size);//创建内核事件表文件描述符
int epoll_ctl(int epfd,//为epoll_create创建内核事件表的句柄
              int op,
              //表示动作，EPOLL_CTL_ADD注册新的fd到epfd;
              //EPOLL_CTL_MOD修改已经注册的fd的监听事件
              //EPOLL_CTL_DEL从epfd删除一个fd
              int fd,//待处理的文件描述符
              struct epoll_event *event);//告诉内核需要监听的事件
//其中event为epoll_event结构体指针类型，表示内核所监听的事件
struct epoll_event{
    __uint32_t events;//epoll_events
    epoll_data_t data;//user data variable
};
events 描述事件类型，其中epoll事件类型有以下几种
EPOLLIN:对应文件描述符可读（包括对端socket正常关闭）
EPOLLOUT:对应文件描述符可写
EPOLLPRI:对应文件描述符有紧急数据可读（带外数据到来）
EPOLLERR:表示对应的文件描述符发生错误
EPOLLHUP:表示对应的文件描述符被挂断
EPOLLET:设置为边缘触发（Edge Triggered）模式，若不设置则默认水平触发,(Level Triggered)
EPOLLONESHOT:只监听一次事件，当监听完这次事件后，如果还要继续监听这个socket，需要再次把该socket加入到EPOLL队列

int epoll_wait(int epfd,//内核事件表句柄
               struct epoll_event* events,//内核事件合集
               int maxevents,//告诉内核该events多大，这个maxevents的值不能大于epoll_create()的size?【待定】
               int timeout)//超时时间，-1阻塞，0立即返回，>0指定毫秒
               返回值：成功返回有多少文件描述符，时间到时返回0，出错返回-1
```

27.几种IO复用的应用场景？

- 当所有fd都是活跃连接时，若使用epoll需要建立文件系统，红黑树和链表。效率不如select 和 poll
- 当监测的fd数目较小，且各fd都比较活跃，最好使用select或poll
- 当监测的fd数目非常大，成千上万，且单位时间内只有一部分就绪，活跃的fd较少，此时用epoll能够提升性能。

28.LT，ET，EPOLLONESHOT区别？

LT水平触发：

- epoll_wait监测到文件描述符有事件发生，则将其通知给应用程序，应用程序可以不立即处理该事件。
- 当下一次调用epoll_wait时，epoll_wait还会再次向应用程序报告此事件，直到被处理。

ET边缘触发：

- epoll_wait监测到文件描述符还有事件发生，则将其通知给应用程序，应用程序必须立即处理该事件
- 必须一次性将数据读取完毕，使用非阻塞IO，读取到eagain

EPOLLONESHOT：

- 一个线程读取某个socket上的数据后开始处理数据，在处理过程中该socket上又有新数据可读，此时另一个线程被唤醒读取，此时出现两个线程处理一个socket
- 若期望一个socket连接在任一时刻都只被一个线程处理，通epoll_ctl对将fd注册为epolloneshot事件，一个线程处理socket，其他线程将无法处理，当该线程处理完后，需要通过epoll_ctl重置epoll_oneshot事件。

29.页式存储，段式存储，段页式存储，段表的逻辑地址和物理地址转换？

https://www.cnblogs.com/zsh-blogs/p/10137909.html

 静态重定位：当用户程序被装入内存时，一次性实现逻辑地址到物理地址的转换，以后不再转换（一般在装入内存时由软件完成），直到该程序完成退出内存为止。

动态重定位：（逻辑地址变换为物理地址是在执行指令时）。

**在段页式虚拟存储管理系统中，假设有如下段表结构信息。**

| 段号 | 基地址 | 段长 |
| ---- | ------ | ---- |
| 0    | 219    | 600  |
| 1    | 2300   | 14   |
| 2    | 90     | 100  |
| 3    | 1327   | 580  |
| 4    | 1952   | 96   |

**请回答下面5个逻辑地址的物理地址分别是多少？**

（1）（0，520）（意思是第0段偏移520个字节，物理地址为基地址219+偏移量520）739

（2）（1，11）  2311   

（3）（2，800）800>100（段的长度），那么直接判定其段号越界

（4）（3，480）1807

（5）（4，156）156>96  段号越界      



**段表地址变换** 

段表：
段号 段长 基址
0 1K 6K
1 600 4K
2 500 8K

求段号为2，偏移量为100的物理地址：

段号2，对应基址为8k，计算：8*1024+100=8292；1k=1024字节



**连续分配方式：**是指为一个用户程序分配一个连续的内存空间。 具体的分为四种方式： 单一连续分配 、固定分区分配 、动态分区分配 、动态重定位分区分配。

 **固定分区分配：**是一种最简单的可运行多道程序的存储管理方式。

**页式存储管理是离散分配方式。**能较好解决外部碎片问题的存储管理方法。

**段式存储管理是离散分配方式。**将作业的地址空间划分为若干个段，每个段定义一组逻辑信息,都有自己的名字,且都是首地址为零、连续的一维线性空间。系统以段为单位分配主存，每一段分配连续的分区。同一进程所包含的各段不要求连续。

**段页式存储**将用户程序分成若干个段（并赋予段名），再把每个段分成若干个页。

  **在未引入快表的分页存储管理时，每读写一个数据，要访问( B )主存。**

A、1次 　　B、2次 　　C、3次 　　D、4次

 若页表全部放在主存，则要取一个数据(一条指令)至少要访问二次主存，第一次是访问页表，确定所取数据(或指令)的物理地址，第二次是根据该地址取数(或指令)。

30.操作系统是一种？（对计算机资源进行管理的系统软件）

31.操作系统基本特征？（并发，共享，虚拟，异步）

32.用户可以通过什么方式使用计算机？

（1）命令方式：命令行方式DOS系统和Unix系统

（2）系统调用方式：（system call）内核提供一系列具备预设功能的函数，通过一组称为系统调用的接口呈现给用户。系统调用将程序请求传给内核，调用响应的内核函数完成所需处理，将结果返回给应用程序。

（3）图标-窗口方式：操作系统提供的图形化界面

33.从资源管理观点来看，操作系统具备五大功能？

(1)处理机管理(2)存储器管理（3）设备管理（4）文件管理（5）操作系统与用户之间的接口

# 多线程多进程总结：

1.进程线程区别？

进程：计算机资源分配的最小单位。线程：CPU处理机调度最小单位。

2.什么是进程上下文，中断上下文？

3.一个进程可以创建多少个线程？和什么有关？

内存大小和ulimit -s 分配栈空间大小。

4.什么时候用多线程？什么时候用多进程？

IO密集型用多线程；CPU计算密集型用多进程。

多线程可用于单机多核；多进程可用于分布式多机。

5.多线程，多进程同步（通信）方法？

多线程同步：互斥锁，信号量（管程），消息队列。【待定】

多进程同步：互斥量，信号量，消息队列，管道，命名管道，socket。

6.进程的空间模型？

.text | .data .bss | 堆   动态连接库 |栈

7.进程线程状态转换图？什么时候阻塞，什么时候就绪，能够直接由阻塞变为就绪吗？

8.父子进程关系及区别?

fork()调用一次，返回两次；返回值<0，调用失败；返回值==0，为子进程；返回值>0为父进程，返回值为子进程的ID。

9.进程私有化什么，公有化什么？线程私有化什么，公有化什么？

- 进程：
- 私有：地址空间，全局变量，堆，栈，计数器，寄存器
- 共享：目录，代码段，文件描述符，进程ID
- 线程：
- 私有：线程栈，寄存器变量，程序计数器
- 共享：地址空间，堆，全局变量，静态变量

10.五种IO模型？

阻塞IO：调用IO函数后必须等待函数返回，期间什么也不做，不停检查函数是否返回，必须等待函数返回才能进行下一步操作。

非阻塞IO：非阻塞等待，每隔一段时间检测IO事件是否就绪，没有就绪就区执行其他事情。非阻塞IO执行系统调用总是立即返回，不管事件是否发生，若事件没有发生则返回-1。此时可以根据errno区分两种情况，对于accept，recv和send，事件未发生时，errno通常被设置为eagain。

IO复用：Linux下使用select/poll/epoll实现IO复用。会使进程阻塞，但和阻塞IO不同的是，可阻塞多个IO操作，而且可以同时对多个读写操作IO函数进行检测，直到有数据可读或可写时，才调用IO操作函数。

信号驱动IO：调用aio_read函数告诉内核描述字缓冲区指针和缓冲区大小，文件偏移及通知的方式，然后立即返回，当内核将数据拷贝到缓冲区后，再通知应用程序。

异步IO。前4种为同步IO。

11.同步和异步？

同步IO是指内核向应用程序通知的就绪事件，比如只通知有客户端连接，要求用户代码自行执行IO操作，异步IO是指内核向应用程序通知的是完成直接，比如读取客户端数据后才通知应用程序，由内核完成IO操作。

若在事件处理模型种：同步指程序按顺序执行；异步需要靠信号驱动执行。

事件处理模式：

（1）reactor：主线程（IO处理单元）只负责监听文件描述符上是否有事件发生，有的话立即通知工作线程（逻辑处理单元），读写数据，接受新连接以及处理客户端请求均在工作线程中完成，通常由同步IO实现。

（2）proactor：主线程和内核负责处理读写数据，接受新连接等IO操作，工作线程仅负责业务逻辑，如处理客户请求，通常由异步实现。

由于异步IO并不成熟，实际可采用同步IO模拟proactor模式：

同步IO模型工作流程（epoll_wait为例）：

- 主线程往epoll内核事件表注册socket上的就绪事件（包括连接事件，读就绪事件）
- 主线程调用epoll_wait等待socket上有数据可读
- 当socket上有数据可读时(fd=|EPOLLIN)，epoll_wait通知主线程，主线程从socket循环读取数据，直到没有更多数据可读，然后将读取到的数据封装成一个请求对象插入请求队列。
- 睡眠在请求队列上的工作线程被唤醒，它获得请求对象并处理客户请求，然后向epoll内核事件表注册socket写就绪事件（EPOLLOUT）
- 主线程调用epoll_wait等待socket可写
- 当socket有数据可写，epoll_wait通知主线程，主线程往socket写入服务器处理客户请求的处理结果。

12.并发和并行？

并发：两个或者多个事件在同一时间间隔内发生。（一段时间内，处理多个事件。）

并行：两个或者多个事件在同一时刻发生。（一个时间点，同时处理多个事件。）

- 并发编程模式：并发编程方式有多线程和多进程两种。在服务器编程中并发模式指IO处理单元与逻辑处理单元协同完成任务的方式。
- （1）半同步/半异步模式
- （2）领导者追随者模式

13.阻塞和非阻塞理解?

阻塞：函数调用返回前不会做其他事情。

非阻塞：函数调用未返回，先做其他事情，一定事件内轮询。

14.如何创建守护进程？

fork()系统调用创建子进程。

父进程exit()退出。

setid()设置子进程会话ID

15.什么是孤儿进程，什么是僵尸进程，怎么处理僵尸进程？

孤儿进程：父进程退出，但子进程未退出，此时子进程为孤儿进程，最终被init1进程收养，并由init1进程对他们完成状态手机工作，由于孤儿进程会被init进程收养，所以孤儿进程不会对系统造成危害。

僵尸进程：子进程退出，子进程的进程描述符在子进程退出时不会释放，只有父进程使用wait()或waitpid()调用回收子进程信息。若子进程退出，但是父进程未调用wait()或waitpid()，子进程的进程描述符仍在系统中，这段时间内，子进程为僵尸进程。【用ps命令显示状态为Z(zombie)】处理僵尸进程先杀死父进程，僵尸进程变为孤儿进程，然后系统回收进程【暂定】。

系统能使用的进程号是有限的，若产生大量僵尸进程，将可能因为没有可用进程号而导致系统不能产生新的进程。

注：

```c++
pid_t wait(int *status);
父进程调用wait()会一直阻塞，直到受到一个子进程退出的SIGCHID信号，之后函数会销毁子进程并返回。若成功则返回被收集的子进程ID，若调用进程没有子进程，调用就会失败，此时返回-1，同时errno被设置为ECHILD。参数status用来保存被收集的子进程退出时的一些状态，如果子进程时如何死掉的毫不在意，指向把这个子进程消灭掉，可以设置这个参数NULL.
    
pid_t waitpid(pid_t pid,int *status,int options);
作用和wait()完全相同，多了两个可由用户控制的参数pid和options；
pid 参数指示一个子进程的ID，表示只关心这个子进程退出时的SIGCHILS信号，如果pid=-1,作用和wait()相同，关心所有子进程退出时的SIGCHILD信号。
options参数主要有WNOHANG和WUNTRACED选项，前者可使调用变为非阻塞的，即立即返回，父进程可继续执行其他任务。
```



```c++
SIGCHILD信号：
当一个子进程改变了它的状态时（停止运行，继续运行或者退出），有两件事会在父进程种发生：
1.得到SIGCHILD信号
2.waitpid()或者wait()调用返回
其中子进程发送的SIGCHILD信号包含了子进程的信息，如进程ID,进程状态，进程使用CPU时间等。
在子进程退出时，它的进程描述符不会立即释放，这是为了让父进程得到子进程的信息，父进程通过wait或waitpid获得退出子进程的信息。
```





16.CPU，内存，虚拟内存，磁盘/硬盘关系？

17.CPU内部结构？

计数器，逻辑运算单元，寄存器

18.半同步/半异步模式工作流程？半同步/半反应堆模式工作流程？

半同步/半异步模式工作流程

> - 同步线程用于处理客户逻辑
> - 异步线程用于处理I/O事件
> - 异步线程监听到客户请求后，就将其封装成请求对象并插入请求队列中
> - 请求队列将通知某个工作在**同步模式的工作线程**来读取并处理该请求对象

半同步/半反应堆工作流程（以Proactor模式为例）

> - 主线程充当异步线程，负责监听所有socket上的事件
> - 若有新请求到来，主线程接收之以得到新的连接socket，然后往epoll内核事件表中注册该socket上的读写事件
> - 如果连接socket上有读写事件发生，主线程从socket上接收数据，并将数据封装成请求对象插入到请求队列中
> - 所有工作线程睡眠在请求队列上，当有任务到来时，通过竞争（如互斥锁）获得任务的接管权

19.池？

池是一组资源的集合，可以是链表，队列，等容器的封装形式，如内存池，线程池，进程池等。改组资源在服务器启动时创建好并初始化，为静态资源。当服务器正式运行阶段，如需要相关资源，直接从池中取，无需动态分配。当处理完事件，将相关资源放回池中，无需执行系统调用释放资源，等待程序运行结束由系统回收。

20.单例模式？两种实现方式，有什么优缺点？

<img src="https://img2018.cnblogs.com/blog/1201066/201903/1201066-20190326083545779-1636922334.png" alt="img" style="zoom:67%;" />

一个类只能有一个实例。

懒汉模式：不用时不去初始化。（非常懒）

饿汉模式：迫不及待地初始化，程序一开始运行就进行初始化。由于静态变量初始化顺序不确定，若在初始化完成之前调用getinstance()可能得到一个未定义的实例造成错误。

```c++
//c11无锁方式：static局部静态变量线程安全的
//懒汉模式
class single{
public:
	static single * getinstance();
private:
    single(){}
    ~single(){}
};
single* single::getinstance(){
    static single obj;
    return &obj;
}
//有锁方式
class single{
public:
	static single* getinstance();
private:
	static pthread_mutex_t lock;
    static single * p;
    ~single(){
        pthread_mutex_init(&lock,NULL);
    }
    single(){}
};
pthread_mutex_t single::lock;
single* single::p=NULL;
single* getinstance(){
    if(p==NULL){
        pthread_mutex_lock(&lock);
        if(p==NULL){
            p=new single;
        }
        pthread_mutex_unlock(&lock);
    }
    return p;
}


//饿汉模式
class single{
public:
   static single* getinstance();
private:
    static single* p;
    single(){}
    ~single(){}
};
single* single::p=new single;
single* single::getinstance(){
    return p;
}

#include<pthread.h>
int main(){
    single* s1=single::getinstance();
    single*	s2=single::getinstance();
    if(s1==s2) cout<<"是同一个实例"<<endl;
    return 0;
}
```

# 操作系统补充

https://www.cnblogs.com/zsh-blogs/p/10125817.html

## 基础部分

(1).多道批处理系统
在单道批处理系统中，内存中仅有一道作业，它无法充分利用系统中的所有资源，致使系统性能较差。 在多道批处理系统中，用户所提交的作业都先存放在外存上并排成一个队列，称为“后备队列”。然后，由作业调度程序按一定的算法从后备队列中选择若干个作业调入内存，使它们共享CPU和系统中的各种资源。

(2).分时系统 

分时系统与多道批处理系统之间有着截然不同的性能差别，它能很好地将一台计算机提供给多个用户同时使用，提高计算机的利用率。分时系统是指，在一台主机上连接了多个带有显示器和键盘的终端，同时允许多个用户通过自己的终端，以交互方式使用计算机，共享主机中的资源。

(3).实时系统 

所谓“实时”，是表示“及时”，而实时系统是指系统能及时响应外部事件的请求，在规定的时间内完成对该事件的处理，并控制所有实时任务协调一致的运行。其应用需求主要在实时控制和实时信息处理。

多道批处理操作系统: 

优点：资源利用率高、系统吞吐量大　　

缺点：平均周转时间长、无交互能力

分时系统的特点：

多路性:同时有多个用户使用一台计算机 宏观上：多人同时使用一个CPU 微观上：多个人在交替使用CPU

交互性:用户根据系统响应结果进一步 提出新请求(用户直接干预每一步)

独占性:用户感觉不到计算机为其他人服务 （OS作为虚机器使各个用户的工作 互不干扰）

及时性:系统对用户提出的请求及时响应

## 进程管理

1.进程与程序本质区别？（静态和动态特征）

2.C源程序到可执行文件的过程？（预处理，编译，汇编，链接）

3.进程的特征？

进程的特征 （1）结构特征： 程序段：进程执行的程序，一般是可重入的“纯代码”。 数据段：程序执行时所用的数据。 PCB： 这三部分构成了进程实体。 （2）动态性（最基本的特征）：由创建而产生，调度而执行，有撤销而消亡 （3）并发性（最重要的特征）：（4）独立性：指进程实体是同一个能独立运行、独立分配资源和独立运行的基本单位。不创建PCB的程序不能作为独立的单位进行运行。 （5）异步性：指进程按各自独立的、不可预知的速度向前推进。

操作系统是根据进程控制块来对并发指向的进程进行控制和管理的。

系统利用进程控制块(PCB)来描述进程的基本情况和活动过程，进而控制和管理进程。系统利用PCB来控制和管理进程，所以PCB是系统感知进程存在的唯一标志。

4.进程状态转换？

当进程处于运行状态时，当等待时间发生时，会由运行转换为**等待状态（阻塞状态）**；而当程序处于等待状态时，若等待发生的事件结束，则由等待转换为就绪。

![img](https://img2018.cnblogs.com/blog/1201066/201812/1201066-20181216155038340-1767517595.png)

5.进程从运行态到等待态可能是（A）。

（PV即信号量的操作，P操作-1，V操作+1）

A．运行进程执行了P操作　　 B．运行进程执行了V操作

C．进程时间片用完　　　　 D．进程被调度

对于A，执行P操作，申请资源，当资源不足时，进程会被阻塞。

对于B，执行V操作，释放资源，是不会阻塞的；

对于C，时间片用完，进程会从执行态转到就绪态，继续等待时间片轮转；

对于D，进程被调度会变为运行态

6.可能导致进程从运行状态变为就绪状态的事件为（  ）。

A．等待I/O操作 　　　　B．等待的事件发生

C．进程运行结束 　　　 D．出现了比当前进程优先级高的进行

进程运行时具有三个状态：运行态，就绪态，等待态（阻塞态）

 A：一次I/O操作结束，等待态->就绪态

 B：运行进程需要I/O操作，运行态->等待态

 C：运行进程结束，直接转入释放

 D：出现了比现运行进程优先级更高的进程，运行态->就绪态

7.必然会引起进程切换的事件为（B）。

A．创建一个新的进程后进入就绪状态 　　B．一个进程从运行状态变为就绪状态

C．一个进程从阻塞状态变为就绪状态 　　D．以上说法都不对

 进程切换是指CPU调度不同的进程执行，当一个进程从运行状态变为就绪状态时，CPU调度另一个进程执行，引起进程切换。

8.用户可以通过（ C）创建或终止一个进程。

A．函数调用 　　B．指令 　　C．系统调用　　 D．命令

9.两个进程合作完成一个任务，在并发执行中，一个进程要等待其合作伙伴发来信息，或者建立某个条件后再向前执行，这种关系是进程间的（ A）关系。

A．同步 　　B．互斥 　　C．合作　　 D．竞争

进程的同步是指并发进程之间存在一种制约关系，一个进程的执行依赖另一个进程的消息，当一个进程没有得到另一个进程的消息时应等待，直到消息到达才被唤醒。所以两个进程合作完成一个任务，在并发执行中，一个进程要等待其合作伙伴发来信息，或者建立某个条件后再向前执行，这种关系是进程间的同步关系。

10.进程间的同步和互斥，分别表示了进程间的（B）。

A．独立和制约 　　B．协作和竞争 　　C．动态性和独立性　　 D．不同状态

11.操作系统在使用信号量解决同步和互斥问题中，若P、V操作的信号量S初始值为3，当前值为-2，则表示有（ 2 ）个等待进程。

 S.value > 0时， S.value为系统中可用资源的数量；

S.value = 0时，可用资源量正好用完；

S.value < 0时，| S.value |为系统中等待使用该资源的队列长度，即 (在信号量上等待的进程数)。

12。执行V操作时，当信号量的值（C），应释放一个等待该信号量的进程。

A．小于0 　　B．大于0 　　C．小于等于0 　　D．大于等于0

pv操作是一对原子操作，p操作的作用是申请资源，即将资源数减１，然后判断资源数是否小于０，若小于０，则自我阻塞在当前资源链表中。

 ｖ操作的作用是归还资源，将所申请的资源数加一，然后判断资源数是否小于等于０，若小于等于０说明有进程阻塞在当前资源上，唤醒一个当前资源链表中的进程。

13.对两个并发进程，其互斥信号量为mutex，若mutex=0,则表明（ C）

A．没有进程进入临界区

B．有两个进程进入临界区

C．一个进程进入临界区但没有进程处于阻塞状态

D．一个进入临界区一个出去等待进入临界区的状态

互斥型信号量是一种特殊的二值信号量，实现对共享资源的独占式处理。它可以在应用程序中用于降解优先级翻转问题。在互斥型信号量中，包括三个元素：1个标志，表示mutex是否可以使用；1个优先级，准备一旦高优先级的任务需要这个mutex，将赋予占有mutex的任务的优先级；1个等待该mutex的任务列表。在互斥型信号量的使用中，其对应的值初始化为1，因此，当其值为1时，所表示的含义是没有进程对这个互斥型信号量所保护的资源进行访问，也就是没有进程进入临界区。临界区不允许两个进程同时进入，D选项明显错误。mutex初值为1，表示允许一个进程进入临界区，当有一个进程进入临界区且没有进程等待进入时，mutex值减1，变为0。

14.下面说法正确的是（ C）。

A．不论是系统支持的线程还是用户级线程，其切换都需要内核的支持

B．线程是资源分配的单位，进程时调度和分派的单位 （调度）

C．不管系统中是否有线程，进程都是拥有资源的独立单位

D．在引入线程的系统中，进程仍是资源调度和分配的基本单位

引入线程后，进程仍然是资源分配的单位。线程是处理器调度和分派的单位，线程本身不具有资源，它可以共享所属进程的全部资源。

15.下列选项中，会导致用户从用户态切换到内核态的操作是（ B）

ⅰ.整数除以零 ⅱ.sin函数调用  ⅲ.read函数调用

A. ⅰ和ⅱ　　 B. ⅰ和ⅲ 　　C. ⅱ和ⅲ 　　D. ⅰ、ⅱ和ⅲ

需要在系统内核态执行的操作是整数除零操作和read系统调用函数，答案选B。

16.进程之间存在哪些制约关系？下列活动属于哪些制约关系？

（1）若干学生去图书馆借同一本书

（2）两队进行篮球比赛

（3）流水线生产的各道工序

（4）商品生产和消费

进程之间存在两种制约关系，即同步和互斥。
同步是由于并发进程之间需要协调完成同一个任务时引起的一种关系，为一个进程等待另一个进程向它直接发送消息或数据时的一种制约关系。
互斥是由于并发进程之间竞争系统的临界资源引起的，为一个进程等待另一个进程已经占有的必须互斥使用的资源时的一种制约关系。
1)是互斥关系，同一本书只能被一个学生借阅，或者任何时刻只能有一个学生借阅一本书。
2)是互斥关系，篮球是互斥资源。
3)是同步关系，一个工序完成后开始下一个工序。
4)是同步关系，生产商品后才能消费。

## 处理机调度与死锁

1、在单处理器的多进程系统中，进程什么时候占有处理器以及决定占用时间的长短是由（ B）决定的。

A、进程运行时间 　　    B、进程的特点和进程调度策略

C、进程执行的代码 　　D、进程完成什么功能

进程调度的时机与进程特点有关，如进程是否为CPU繁忙型还是I/O繁忙型、自身的优先级等。但是仅这些特点是不够的，能否得到调度还取决于进程调度策略，若采用优先级调度算法，则进程的优先级才起作用。至于占用处理器运行时间的长短，则要看进程自身，若进程是I/O繁忙型，运行过程中要频繁访问I/O端口，也就是说，可能会频繁放弃CPU。所以，占用CPU的时间就不会长，一旦放弃CPU，则必须等待下次调度。若进程是CPU繁忙型，则一旦占有CPU就可能会运行很长时间，但是运行时间还取决于进程调度策略，大部分情况下，交互式系统为改善用户的响应时间，大多数采用时间片轮转的算法，这种算法在进程占用CPU达到一定时间后，会强制将其换下，以保证其他进程的CPU使用权。所以选择B选项。

2、时间片轮转算法是为了（ A）

A、多个用户能及时干预系统 　　B、优先级较高的进程能得到及时响应

C、是系统变得更为高效 　　　　D、需要CPU时间最少的进程最先执行

时间片轮转的主要目的是使得多个交互的用户能够得到及时响应，使得用户以为“独占”计算机的使用。因此它并没有偏好，也不会对特殊进程做特殊服务。时间片轮转增加了系统开销，所以不会使得系统高效运转，吞吐量和周转时间均不如批处理。但是其较快速的响应时间使得用户能够与计算机进行交互，改善了人机环境，满足用户需求。

3，（B）有利于CPU繁忙型的作业，而不利于I/O繁忙型的作业。

A、时间片轮转算法 　　B、先来先服务调度算法

C、短作业优先算法　　 D、优先级调度算法

先来先服务(FCFS)调度算法是一种最简单的调度算法，当在作业调度中采用该算法时，每次调度是从后备作业队列中选择一个或多个最先进入该队列的作业，将它们调入内存，为它们分配资源、创建进程，然后放入就绪队列。
FCFS调度算法比较有利于长作业，而不利于短作业。所谓CPU繁忙型的作业，是指该类作业需要大量的CPU时间进行计算，而很少请求I/O操作。I/O繁忙型的作业是指CPU处理时，需频繁的请求I/O操作。所以CPU繁忙型作业更接近于长作业。答案选择B选项。

4、为了照顾短作业用户应采用（ B  ）调度算法；为了能实现人机交互应采用（ D ）调度算法；既能使短作业用户满意又能使长作业用户满意应采用（ C  ）调度算法。

A、FCFS(先来先服务) 　　B、SJF(短作业优先) 　　C、HRRN(高响应比优先)　　 D、RR(时间片轮转)

照顾短作业用户，选择短作业优先调度算法(SJF)；照顾紧急作业用户，即选择优先级高的作业优先调度，采用基于优先级的剥夺调度算法()；实现人机交互，要保证每个作业都能在一定时间内轮到，采用时间片轮转法(RR)；使各种作业用户满意，要处理多级反馈，所以选择多级反馈队列调度算法。

5、有三个作业分别为J1、J2、J3，其运行时间分别为2h、5h、3h，假定它们能同时达到，并在同一台处理器上以单道方式运行，则平均周转时间最小的执行顺序为（    J1,J3,J2     ）

本题目考查平均周转时间的计算。(J1,J3,J2)所对应的平均周转时间为(2+2+3+2+3+5)/3=17/3。

![img](https://img2018.cnblogs.com/blog/1201066/201812/1201066-20181217160050870-501849093.png)![img](https://img2018.cnblogs.com/blog/1201066/201812/1201066-20181217160118399-255293751.png)

6、关于优先权大小的论述中，正确的是（ D ）。

A、资源要求多的作业优先权应高于资源要求少的作业优先权

B、用户进程的优先权，应高于系统进程的优先权

C、在动态优先权中，随着作业等待时间的增加，其优先权将随之下降

D、在动态优先权中，随着作业执行时间的增加，其优先权将随之下降

系统进程的优先权应高于用户进程的优先权。作业的优先权与长作业、短作业或者是系统资源要求的多少没有必然的关系。在动态优先权中，随着进程执行时间的增加其优先权随之降低，随着作业等待时间的增加其优先权应上升。

7、进程调度算法采用固定时间片轮转调度算法，当时间片过大时，就会使时间片轮转算法转化为（ B）调度算法。

A、HRRN 　　B、FCFS 　　C、SPF 　　D、优先级

时间片轮转调度算法在实际运行中也是按先后顺序使用时间片，当时间片过大时，我们可以认为其大于进程需要的运行时间，即转变为先来先服务调度算法。

8、在调度算法中，对短进程不利的是（  B ）调度算法。

A、SPF 　　B、FCFS 　　C、HRRN 　　D、多级反馈队列

先来先服务调度算法中，若一个长进程(作业)先到达系统，就会使后面许多短进程(作业)等待很长时间，因此对短进程(作业)不利。

9、下列选项中，满足短任务优先且不会发生饥饿现象的调度算法是（  B ），最有利于提高系统吞吐量的调度算法是（  D ）。

A、FCFS 　　B、 HRRN 　　C、 RR 　　D、SJ(P)F

由于响应比等于等待时间加上服务时间再除以服务时间，所以等待时间相同时，短作业的响应比更大，能优先获得时间片，另一方面，当服务时间相同时，等待时间越长，响应比越大，所以同时照顾了长作业。

10、下列调度算法中，下列选项中，不可能导致饥饿现象的调度算法是（  A）。

A、RR 　　B、静态优先数调度　　C、非抢占式短作业优先 　　D、抢占式短作业优先

采用静态优先级调度时，当系统总是出现优先级高的任务时，优先级低的任务会总是得不到处理机而产生饥饿现象；而短任务优先调度不管是抢占式或是非抢占的，当系统总是出现新来的短任务时，长任务会总是得不到处理机，产生饥饿现象，因此B、C、D都错误，选A。

11、一个进程的读磁盘操作完成后，操作系统针对该进程必做的是（   A）。

A、修改进程状态为就绪态 　　    B、降低进程优先级

C、给进程分配用户内存空间 　　D、增加进程时间片大小

进程申请读磁盘操作的时候，因为要等待I/O完成，将自身阻塞，进入阻塞态。当 I/O完成之后，从阻塞进入就绪态

12、采用时间片轮转调度算法分配CPU时，当处于运行状态的进程用完一个时间片后，它的状态是（ C）状态。

A、阻塞 　　B、运行　　 C、就绪 　　D、消亡

处于运行状态的进程用完一个时间片后，它的状态会变为就绪状态等待下一次处理器调度。当进程执行完最后的语句并使用系统调用exit，请求操作系统删除它或出现一些异常情况时，进程才会终止。

13、下列情况会导致系统发生死锁的是（  C ）。

A、进程释放资源 　　　　　　　　　　    B、一个进程进入死循环

C、多个进程竞争资源出现了循环等待 　　D、多个进程竞争使用共享型的设备

死锁：是指多个进程在运行过程中因争夺资源而造成的一种僵局，当进程处于这种僵持状态时，若无外力作用，它们都将无法再向前推进（即都不能继续执行）。 定义中须注意的几点： 多个进程：只有多个进程在同时运行时才可能 出现争夺资源的情况； 争夺资源：有限资源； 僵持状态：无休止的等待。

两个或两个以上并发进程，如果每个进程持有某种资源，而又等待着别的进程释放它或它们现在保持着的资源，否则就不能向前推进，此时，每个进程都占用了一定的资源，但又都不能向前推进。这种现象称为死锁。
死锁的起因：(1)互斥条件；(2)不可剥夺条件；(3)部分分配；(4)环路条件。

14、对资源采用按序分配策略能达到（  A  ）的目的。

A、预防死锁 　　B、避免死锁　　 C、检测死锁 　　D、解除死锁

对于死锁的预防可以采取3种措施：

采用资源的静态预分配策略，破坏“部分分配”条件；

允许进程剥夺使用其它进程占有的资源，从而破坏“不可剥夺”条件；

采用资源有序分配法，破坏“环路”条件。

15、死锁预防是保证系统不进入死锁状态的静态策略，其解决办法是破坏产生死锁的4个必要条件之一，下列办法中破坏了“循环等待”条件的是（ D ）。

A、银行家算法 　　B、一次性分配策略 　　C、剥夺资源法 　　D、资源有序分配策略

循环等待是死锁的一个条件，一个确保此条件不成立的方法是对所有的资源类型进行完全排序，且要求每个进程按递增顺序来申请资源。

破坏“请求和保持”条件，采取一次性分配策略（也叫静态分配策略）。或者允许进程只获得运行初期所需资源，运行过程中逐步释放已用完资源，然后再请求新的资源。

银行家算法是避免死锁算法。

16、银行家算法是一种（ B）算法。

A、预防死锁　　B、避免死锁 　　C、检测死锁 　　D、解除死锁

银行家算法是一种最有代表性的避免死锁的算法。在避免死锁方法中允许进程动态地申请资源，但系统在进行资源分配之前，应先计算此次分配资源的安全性，若分配不会导致系统进入不安全状态，则分配，否则等待。

17、在下列解决死锁的方法中，属于死锁预防策略的是（ B ）。

A、银行家算法 　　　　B、一次性分配策略

C、资源分配图简化 　　D、死锁检测

在解决死锁的方法中，死锁的预防是设法减少破坏产生死锁的必要条件之一。银行家算法属于死锁的避免，不很严格地限制产生死锁的必要条件的存在，而是在系统运行过程中小心地避免死锁的最终发生。死锁检测算法，允许死锁发生，定期检测。所以，只有资源有序分配法属于预防死锁的策略。

18、某系统有n台互斥使用的同类设备，三个并发进程分别需要 3、4、5 台设备，可确保系统不发生死锁的设备数 n 最小为（ B ）。

A．9 　　B．10 　　C．11　　 D．12

极端状态下：

 进程1(3台)：申请到2台，无法工作；

 进程2(4台)：申请到3台，无法工作；

 进程3(5台)：申请到4台，无法工作；

 申请总数：2+3+4=9，此时若只有9台，3个进程持续申请且申请不到，造成死锁。

 所以必须再空出一台。

 19、某计算机系统中有 8 台打印机，由 K 个进程竞争使用，每个进程最多需要 3 台打印机。该系统可能会发生死锁的 K 的最小值是（ C ）。

A．2 　　B．3 　　C．4　　 D．5

 最多每个进程需要3台。先实际分配给每个进程2台。设最多X台不死锁。有如下等式：

 2*X + 1 <= 8   得出 X=3

 题目问的是：最少多少个进程使得会发生死锁。

 故 X+1 = 4 个进程。

 20、死锁与安全状态的关系是（ D）。

A．死锁状态有可能是安全状态 　　B．安全状态有可能成为死锁状态

C．不安全状态就是死锁状态 　　    D．死锁状态一定是不安全状态

并非所有的不安全状态都是死锁状态，但当系统进入不安全状态后，便可能进入死锁状态；反之，只要系统处于安全状态，系统便可以避免进入死锁状态；死锁状态必定是不安全状态。

21、下列关于死锁的说法正确的是（ D）。

I.死锁状态一定是不安全状态

II.系统资源分配不足和进程推进顺序非法是产生死锁的原因

III.资源的有序分配策略可以破坏死锁的循环等待条件

IV.采用资源剥夺方法可以解除死锁，还可以采用撤销进程方法解除死锁

A．Ⅰ 　　B．Ⅰ、Ⅱ 　　C．Ⅰ、Ⅲ 　　D．Ⅰ、Ⅱ、Ⅲ、Ⅳ

## 存储管理

一、单项选择题

1、存储管理的目的是( C  ) 。

A、方便用户              　　　　  B、提高主存空间利用率 

C、方便用户和提高主存利用率  D、增加主存实际容量

存储管理的目的有两个：一个是方便用户，二是提高内存利用率。

2、存储器管理中，下列说法正确的是（ B ）

A、无论采用哪种存储管理方式，用户程序的逻辑地址均是连续的

B、地址映射需要有硬件支撑地址变换

C、段表和页表都是由用户根据进程情况而建立的

D、采用静态重定位可实现程序浮动

 静态重定位 当用户程序被装入内存时，一次性实现逻辑地址到物理地址的转换，以后不再转换（一般在装入内存时由软件完成），直到该程序完成退出内存为止。

动态重定位（逻辑地址变换为物理地址是在执行指令时）。

 3、动态重定位是在作业的( D )中进行的。

A、编译过程  　　B、装入过程 　　C、修改过程 　　D、执行过程

动态重定位是在作业运行时执行到一条访存指令时再把逻辑地址转换为主存中的物理地址，实际中是通过硬件地址转换机制实现的。

动态重定位（逻辑地址变换为物理地址是在执行指令时）。

4、(  A )要求存储分配时具有连续性。

A、固定分区存储管理　　 B、页式存储管理 

C、段式存储管理 　　　　D、段页式存储管理

 连续分配方式：是指为一个用户程序分配一个连续的内存空间。 具体的分为四种方式： 单一连续分配 、固定分区分配 、动态分区分配 、动态重定位分区分配。

 固定分区分配是一种最简单的可运行多道程序的存储管理方式。

页式存储管理是离散分配方式。能较好解决外部碎片问题的存储管理方法。

段式存储管理是离散分配方式。将作业的地址空间划分为若干个段，每个段定义一组逻辑信息,都有自己的名字,且都是首地址为零、连续的一维线性空间。系统以段为单位分配主存，每一段分配连续的分区。同一进程所包含的各段不要求连续。

段页式存储将用户程序分成若干个段（并赋予段名），再把每个段分成若干个页。

5、(   C )存储管理支持多道程序设计，算法简单，但内部碎片多。

A、段式  　　B、页式  　　C、固定分区 　　D、段页式

固定分区分配是最简单的多道程序的存储管理方式。在此方式中，由于每个分区的大小固定，必然会存储空间的浪费。

6、提高主存利用率主要是通过( A )实现的。

A、内存分配 　　B、内存保护 　　C、地址转换　　 D、内存扩充

7、动态分区管理方式按作业需求量分配主存分区，所以( D )。

A、分区的长度是固定

B、分区的个数是确定的

C、分区长度和个数都是确定

D、分区的长度不是预先固定的，分区的个数是不确定的

 可变分区分配 方法：系统不预先建立分区，分区的建立是在作业处理时进行，这样做的目的是使分区的大小正好满足用户作业的需要，分区的大小及个数都是不固定的。

 8、( A )存储管理不适合多道程序系统。

A、单一连续分配 　　B、固定分区 　　C、可变分区　　 D、段页式

单一连续分配管理方式只能适用于单用户、单任务的操作系统中，不适合多道程序设计。内存利用率很低。

9、碎片现象的存在使(   A  )。

A、主存空间利用率降低    　　B、主存空间利用率提高

C、主存空间利用率得以改善    D、主存空间利用率不受影响

由于碎片现象，使得部分内存因为太小而不能被利用，使得内存空间利用率降低。

10、较好地解决了外部碎片问题的存储管理方法是（D）。

A、动态分区管理 　　B、段式存储管理

C、固定分区管理 　　D、页式存储管理

 分页存储管理器方式的优缺点： 优点：由于这种内存分配方式不要求程序或进程的程序段和数据在内存中连续存放，消除了外部碎片，从而能在一定程度提高内存的利用率，又有利于组织多道程序执行。 缺点：易产生页内碎片

11、下列选项中，不会产生内部碎片问题的存储管理方法是（B）。

A、分页存储管理 　　B、分段存储管理

C、固定分区存储管理 D、段页式存储管理

段则是信息的逻辑单位，它含有一组其意义相对完整的信息。 分段的目的是为了能更好地满足用户的需要。段的长度却不固定， 决定于用户所编写的程序，通常由编译程序在对源程序进行编译时，根据信息的性质来划分。

12、最佳适应分配算法把空闲区( C )。 

A、按地址递增顺序登记在空闲区表中 

B、按地址递减顺序登记在空闲区表个 

C、按长度以递增顺序登记在空闲区表中 

D、按长度以递减顺序登记在空闲区表中

 最佳适应算法 方法：为作业选择分区时总是寻找其大小最接近于作业所要求的存储区域的分区。 特点：用最小空间满足要求，较大的空闲区被保留，有利于满足长作业的 要求。 缺点：分配后剩下的空闲区难以满足别的用户作业的需要。最佳适应算法要求从剩余的空闲分区中选出最小且满足存储要求的分区，空闲区应按长度递增登记在空闲区表中。

 13、某基于动态分区存储管理的计算机，其主存容量为 55MB（初始为空闲），采用最佳适配算法，分配和释放的顺序为：分配 15MB，分配 30MB，释放 15MB，分配 8MB，分配 6MB，此时主存中最大空闲分区的大小是（B）。

A、7MB 　　B、9MB　　 C、10MB 　　D、15MB

最佳适配算法是指：每次为作业分配内存空间时，总是找到能满足空间大小需要的最小的空闲分区给作业。可以产生最小的内存空闲分区。

下图显示了这个过程的主存空间的变化。
![img](https://gss0.baidu.com/7LsWdDW5_xN3otqbppnN2DJv/doc/pic/item/a71ea8d3fd1f41344446e9a52d1f95cad0c85efc.jpg)
图中，灰色部分为分配出去的空间，白色部分为空闲区。这样，容易发现，此时主存中最大空闲分区的大小为9MB。

 14、在未引入快表的分页存储管理时，每读写一个数据，要访问( B )主存。

A、1次 　　B、2次 　　C、3次 　　D、4次

 若页表全部放在主存，则要取一个数据(一条指令)至少要访问二次主存，第一次是访问页表，确定所取数据(或指令)的物理地址，第二次是根据该地址取数(或指令)。

 15、动态分区存储管理的( D )总是按作业要求挑选一个最大的空闲区。

A、顺序分配算法 　　 B、最先适应分配算法

C、最优适应分配算法  D、最坏适应分配算法 

首次适应法： 方法：为作业选择分区时总是按地址从低到高搜索，只要找到可以容纳该作业的空白块，就把该空白块分配给该作业。 特点：先利用低址部分的空闲区，保存了高址的大空闲区，为大作业分配创造了条件 缺点：低址部分被不断划分，有很多碎片；每次都从低址部分查找，增加查找开销。

循环首次适应法 方法：类似首次适应法每次分区时，总是从上次查找结束的地方开始，找到一个足够大的空白区分配。 特点：空闲区域分布得均匀，减少查找开销 缺点：缺乏大的空闲区域。

最佳适应算法 方法：为作业选择分区时总是寻找其大小最接近于作业所要求的存储区域的分区。 特点：用最小空间满足要求，较大的空闲区被保留，有利于满足长作业的 要求。 缺点：分配后剩下的空闲区难以满足别的用户作业的需要。

最坏适应算法 方法：与最佳适应法相反，它在作业选择存储块时，总是寻找最大的空白区。 特点：当分割后空闲块仍为较大空块 缺点：空闲区均匀减少，工作一段时间后，难以满足大作业的需要。

16、一个分段存储管理系统中，地址长度为 32 位，其中段号占 8 位，则最大段长是 （）。

A、2^8  B 　　B、2^16B 　　C、2^24B 　　D、2^32B

段地址为32位二进制数，其中8位表示段号，则段内偏移量占用32-8=24位二进制数，故最大段长为2^24B。

17、抖动是指( B )。

A、使用机器时，造成屏幕闪烁的现象

B、刚被调出的页面又立即被装入所形成的频繁装入\调出的现象

C、系统盘有问题，造成系统不稳定的现象

D、由于主存分配不当，偶然造成主存不够的现象

不适当的算法可能会导致进程发生抖动，即刚被换出的页很快又被访问需要将它重新调入，如此频繁的页面更换，以致一个进程在运行中把大部分时间都花费在页面置换工作上，我们称进程发生了“抖动”。

18、虚拟存储管理系统的基础是程序的( B  )理论。

A、动态性 B、全局性

C、局部性 D、虚拟性

19、在段式存储管理中，(   C )。

A、段间绝对地址一定不连续

B、段间逻辑地址必定连续

C、以段为单位分配，每段分配一个连续主存区

D、每段是等长的

 基本段式管理方式 将作业的地址空间划分为若干个段，每个段定义一组逻辑信息,都有自己的名字,且都是首地址为零、连续的一维线性空间。系统以段为单位分配主存，每一段分配连续的分区。同一进程所包含的各段不要求连续. 分配方式：离散分配

 20、虚拟存储技术不能以( A )为基础。

A、分区存储管理 　　B、段式存储管理 　　C、页式存储管理 　　D、段页式存储管理

虚拟存储技术是将内存和外存结合起来管理，为用户提供一个比内存空间大得多的虚拟存储器。其思想是：当进程要求执行时，将它的一部分程序或数据调入内存，另一部分暂时存放在外存，进程在运行时，如果要使用的信息不在内存时，发中断，由系统将它们从外存调入内存。虚拟存储管理分为虚拟页式、虚拟段式和虚拟段页式。在分区管理中，可以通过覆盖和交换技术来扩充内存，但由于各个进程对应不同的分区以及在分区内各个进程连续连续存放，进程的大小仍然受分区大小或内存可用空间的限制，不能实现虚拟存储。

21、在动态分区存储管理中的拼接技术可以（A）。

A、集中空闲区 　　　B、增加主存容量

C、缩短访问周期 　　D、加速地址转换

 动态重定位分区分配方式= “紧凑”技术+重定位+动态分区分配方式。

可重定位分区的优缺点 优点：解决了可变分区分配所引入的“外零头”问题。消除内存碎片，提高内存利用率。 缺点：提高硬件成本，紧凑时花费CPU时间。

 22、在分页系统环境下，程序员编制的程序，其地址空间是连续的，分页是由（  D  ）完成的。

A、程序员 　　B、编译地址 　　C、用户 　　D、系统

分页是由操作系统自动完成的，一个操作系统一旦设计完成，其存储管理系统的结构就已经确定，分页还是分段，页面大小等在设计操作系统的过程中已经确定，当一个程序被创建为进程，并分配资源，其页面的大小自动分割完成，对用户是透明的，对编译程序和链接装配程序透明(在相同的系统里)。只有操作系统可以感知页面的存在，在内存管理过程中，操作系统要为用户进程分配内存，回收内存。所以操作系统是页面最直接的接触者，它将页面从计算机系统中到用户(包括程序员)进行了隔离。

23、下列关于虚拟存储器的叙述中，正确的是（  B   ）。

A．虚拟存储只能基于连续分配技术 　　    B．虚拟存储只能基于非连续分配技术

C．虚拟存储容量只受外存容量的限制 　　D．虚拟存储容量只受内存容量的限制

虚拟存储器只能基于非连续分配技术。虚拟存储容量是虚拟的空间，与逻辑地址的位数相关，不会只受到内存或外存容量的限制。

24、请求分页系统中的页表项中，访问位供（ D ）时参考。

A、分配页面  　　B、置换算法  　　C、程序访问 　　 D、换出页面

![img](https://img2018.cnblogs.com/blog/1201066/201812/1201066-20181218154341970-343248326.png)

状态位P： 指示该页是否已调入内存，以供程序在运行时参考；

访问字段A：记录该页在一段时间内被访问的次数，或记录该页最近有多长时间未被访问，以供系统在换出页面时参考。

修改位M： 表示该页在调入内存后是否被修改过。

外存地址：指出该页在外存上的地址，通常是物理块号/盘块号。

25、请求分页系统中的页表项中，外存地址供（   D  ）时参考。

A、分配页面  　　B、调入页面  　　C、程序访问 　　 D、换出页面

26、在段页式存储管理系统中，内存等分成（A），程序按逻辑模块划分成若干（D）。

A、块 　　B、分区 　　C、段长 　　D、段

27、下述（  A  ）页面淘汰算法会产生Belady现象。

A、先进先出　　B、最近最少使用 　　C、最近最久未使用 　　D、最佳

 先进先出(FIFO)页面置换算法 系统将最先进入到内存中的页面换出到外存的对换区中，即选择在内存中停留时间最长的的页面予以淘汰。

这种算法比较容易实现，系统只要将进程在内存中的页面按照进入内存时间的先后顺序排序，并组织成一个队列，另外再设置一个指针，使这个指针总执行最早进入到内存的那个页面。 性能差，页面调入的先后并不能反映页面的使用情况。它有一种异常现象，即在增加存储块的情况下，反而使缺页中断率增加了。

 28、考虑一个分页系统，其页表存放在内存。

（1）如果内存读写周期为1.0微秒，则CPU从内存取一条指令或一个操作数需时间为（D） 微秒。

 因为页表放在内存，故取一条指令（或一个操作数）须访问两次内存，所以需1.0us×2 = 2.0us的时间。

 （2）如果设立一个可存放8个页表项的快表，80%的地址变换可通过快表完成，内存平均存取时间为（C）微秒。（假设快表访问时间可忽略）

![img](https://img2018.cnblogs.com/blog/1201066/201812/1201066-20181218160818484-409041138.png)

A、1.0 　　B、1.1 　　C、1.2 　　D、2.0
