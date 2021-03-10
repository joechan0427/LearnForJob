# BIO\NIO\同步IO\异步IO
IO操作分2个步骤，请求IO和实际IO
**请求 IO :** 进程向内核发出 IO 请求, 内核返回数据报文是否已准备好
**实际 IO :** 从内核向进程复制数据

**举例 :** 对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所有等待分组到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用程序缓冲区
> 用户空间和内核空间
    1. 操作系统为了支持多个应用同时运行，需要保证不同进程之间相对独立（一个进程的崩溃不会影响其他的进程 ， 恶意进程不能直接读取和修改其他进程运行时的代码和数据）。 因此操作系统内核需要拥有高于普通进程的权限， 以此来调度和管理用户的应用程序。
    2. 于是内存空间被划分为两部分，一部分为内核空间，一部分为用户空间，内核空间存储的代码和数据具有更高级别的权限。内存访问的相关硬件在程序执行期间会进行访问控制（ Access Control），使得用户空间的程序不能直接读写内核空间的内存

==阻塞, 非阻塞是针对第一阶段而言
同步, 非同步是针对第二阶段而言==

1. 阻塞IO
![](https://pic2.zhimg.com/80/e83d68da03da2e8c1568b4b4b630edfd_1440w.jpg?source=1940ef5c)

2. 非阻塞IO
![](https://pic1.zhimg.com/80/4bc31cab27a9a732ab7d1ba9e674ed64_1440w.jpg?source=1940ef5c)
进程把一个套接字设置成非阻塞是在通知内核，当所请求的I/O操作非得把本进程投入睡眠才能完成时，不要把进程投入睡眠，而是返回一个错误。==recvfrom总是立即返回==

3. IO 多路复用
![](https://pic2.zhimg.com/80/b1ec6a4f16844a27c175d5a6a94cd7f8_1440w.jpg?source=1940ef5c)
I/O多路复用的函数也是阻塞的，但是其与以上两种还是有不同的，I/O多路复用是阻塞在select，epoll这样的系统调用之上，而没有阻塞在真正的I/O系统调用如recvfrom之上。

4. 异步 IO
![](https://pic1.zhimg.com/80/5819fd0fdff2bd4fdc9652291aca1831_1440w.jpg?source=1940ef5c)
工作机制是告知内核启动某个操作，并让内核在整个操作（包括将数据从内核拷贝到用户空间）完成后通知我们

## 阻塞原理
阻塞是指进程在发起了一个系统调用（System Call） 后， 由于该系统调用的操作不能立即完成，需要等待一段时间，于是内核将进程挂起为等待 （waiting）状态， 以确保它不会被调度执行， 占用 CPU 资源

> 系统调用（system call）
    system call 是操作系统提供给应用程序的接口。 用户通过调用 systemcall 来完成那些需要操作系统内核进行的操作， 例如硬盘， 网络接口设备的读写等。

**工作队列:**
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221075-ac2ec9f0-c364-40a9-a8d6-ec7abae56599.jpeg)
操作系统为了支持多任务, 把进程分为 "运行" 和 "等待" 等几种状态
如上图的进程 A, 在执行到 recv 整个 IO 阻塞方法时, 操作系统会将其转换为 "等待" 状态, 等到接收到数据, 再转换为 "运行" 的状态

**等待队列**
当进程 A 执行创建 socket 的语句时, 操作系统会创建一个由文件系统(图中的文件列表)管理的 socket 对象
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221524-91520e15-e756-4a79-988c-cf8be8068f90.jpeg)
一个 socket 对象包括收发缓冲区, 和等待列表, 当进程 A 执行到 recv 方法的时候, 会把该进程放入对应的 socket 对象的等待队列中(==对于阻塞 IO 来说==)
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221078-f8a161c0-5ca2-47dc-ba2b-c044e41c7de1.jpeg)


## IO 多路复用
### select
select 的伪代码
```c
int s = socket(AF_INET, SOCK_STREAM, 0); 
bind(s, ...) 
listen(s, ...) 
int fds[] = 存放需要监听的socket 
while (1) { 
    int n = select(..., fds, ...) 
    for(int i=0; i < fds.count; i++){ 
        if(FD_ISSET(fds[i], ...)){ 
            //fds[i]的数据处理 
        } 
    } 
}
```
#### select 的流程
1. 当进程 A 需要监听多个 socket 的时候, 便把该进程加入到需要监听的 socket 的等待队列里
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221394-15a15def-7d6a-40b9-9389-64dff4a0c68a.jpeg)
2. 当其中一个 socket 接收到数据, 操作系统将唤醒在该 socket 的等待队列中的线程 A
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221471-7df6eef9-f66e-4729-8f1a-773e465492c9.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_eXVsb25nc3Vu%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)
3. 线程A 被唤醒后, 由于不知道是哪个 socket 接收到数据, 因此需要遍历一遍 socket 列表, 即伪代码里的 fds

**缺点:**
1. 每次唤醒后, 需要遍历一遍 socket 列表来确认哪个 socket 对象接收到数据(因此, 规定最大只能监控 1024 个 socket 对象)
2. 每次执行完, 需要重新将进程A 加入到各个 socket 对象的等待队列中

#### epoll
```c
int s = socket(AF_INET, SOCK_STREAM, 0); 
bind(s, ...) 
listen(s, ...) 
int epfd = epoll_create(...); 
epoll_ctl(epfd, ...); //将所有需要监听的socket添加到epfd中 
while(1) { 
    int n = epoll_wait(...) 
    for(接收到数据的socket){ 
        //处理 
    } 
}
```
#### epoll 流程
1. 创建 epoll 对象, 当进程A 执行 epoll_creat 方法时, 内核会创建一个 eventpoll 对象(伪代码里的 epfd)
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221115-cdca633e-0069-4b49-aac7-884c5e279f25.jpeg)
eventpoll 对象也是文件系统的一员
2. 维护监视列表
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221111-214398cf-4adc-479b-aeeb-2351cfc315a6.jpeg)
==内核会将 eventpoll 添加到进程 A 需要监听的 Socket 的等待队列中==
当 Socket 收到数据后，中断程序会操作 eventpoll 对象，而不是直接操作进程
3. 接收数据
当 socket 接收到数据之后, 中断程序会给 eventpoll 对象中的就序列表(rdlist) 添加 socket 引用
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221169-416a5e91-600c-47f2-b004-0f6a2c192e71.jpeg)
4. 阻塞和唤醒进程
当进程 A 运行到 epoll_wait() 方法时, 操作系统会把进程A 放在 eventpoll 对象的等待队列里, 然后阻塞进程 A
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221241-a82b67e8-8b22-4b1a-aaea-da9f5b333596.jpeg)
当 socket 接收到数据, 中断程序一方面将 socket 放入 rdlist, 另一方面将 eventpoll 中的进程 A 唤醒进入运行状态
![](https://cdn.nlark.com/yuque/0/2020/jpeg/181910/1608369221108-acb69d23-01d1-4a50-a177-84193b28effd.jpeg?)
[io模型](https://www.jianshu.com/p/486b0965c296)
[select、poll、epoll详解](https://www.jianshu.com/p/dfd940e7fca2)