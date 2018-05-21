

##part32 socket并发编程

[TOC]

###socket原理


应用程序A(用户内存)->A内核: 发送 data

​	A内核- >网络：network

​	网络- >B内核：接收data

​	B内核->B程序(用户)

​	



[老男孩课程笔记](http://www.cnblogs.com/linhaifeng/articles/6129246.html)

[](http://www.cnblogs.com/linhaifeng/articles/6129246.html)

###1. udp vs tcp区别

- udp 无连接 
  - SOCK_DGRAM
  - 不可靠传输
  - 基于数据报文
- tcp  有连接 ,必须先启动服务器，然后启动客户端去连接服务端
  - back_log 半连接池
  - SOCK_STREAM
  - 可靠传输
  - 基于数据流(btyles)



###2. udp vs tcp 共同点

- 字节(bytes)传输(需要编码)



###3. 模块subprocess使用

- Popen('cmd',shell=True,stdout=subprocess.PIPE).stdout.read() 通过管道读取，能取一次
- stdout
- stdin
- stderr

###4.黏包现象 

- 原因：
  - 每次发送数据比较小，tcp会通过算法合并数据发送，而另一端无法区别消息边界。
  - 发送的消息太大，另一端接受的最大消息小于发送消息大小。
- 只有tcp有此现象，udp没有
- recv(tcp)在自己端的缓冲区为空时，阻塞。recvfrom(udp)在自己端的缓冲区为空时，收一个空值。
- tcp send和recv 和udp sendfrom和recv不一样，区别是udp每次send数据，都会recv一个完整的send数据，所以说udp是有消息边界的，而tcp不一样，如果传输的数据小，tcp会通过算法合并，而且接受是按照buffersize接受，无法区分消息边界。
- 解决方案：low方案

####服务器端

```

#
__author__ = 'Linhaifeng'
import socket,subprocess
ip_port=('127.0.0.1',8080)
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

s.bind(ip_port)
s.listen(5)

while True:
    conn,addr=s.accept()
    print('客户端',addr)
    while True:
        msg=conn.recv(1024)
        if not msg:break
        res=subprocess.Popen(msg.decode('utf-8'),shell=True,\
                            stdin=subprocess.PIPE,\
                         stderr=subprocess.PIPE,\
                         stdout=subprocess.PIPE)
        err=res.stderr.read()
        if err:
            ret=err
        else:
            ret=res.stdout.read()
        data_length=len(ret)
        conn.send(str(data_length).encode('utf-8'))
        data=conn.recv(1024).decode('utf-8')
        if data == 'recv_ready':
            conn.sendall(ret)
    conn.close()

​```

####客户端

​```

    author = 'Linhaifeng'

    import socket,time

    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)

    res=s.connect_ex(('127.0.0.1',8080))

    while True:

      msg=input('>>: ').strip()
      if len(msg) == 0:continue
      if msg == 'quit':break

    s.send(msg.encode('utf-8'))
    length=int(s.recv(1024).decode('utf-8'))
    s.send('recv_ready'.encode('utf-8'))
    send_size=0
    recv_size=0
    data=b''
    while recv_size < length:
        data+=s.recv(1024)
        recv_size=len(data)


    print(data.decode('utf-8'))
```

- 高大上方案：
  - struct.pack('i',123) 该模块可以把一个类型，如数字，转成固定长度的bytes
  - [struct使用文档](http://www.cnblogs.com/coser/archive/2011/12/17/2291160.html)
  - ![struck使用说明](https://images2015.cnblogs.com/blog/1036857/201704/1036857-20170422071900493-2119801952.png)



###5.IP端口被占用解决措施

>有的同学在重启服务端时可能会遇到
>
>![](https://images2015.cnblogs.com/blog/1036857/201701/1036857-20170101090541757-870871601.png)
>
>这个是由于你的服务端仍然存在四次挥手的time_wait状态在占用地址（如果不懂，请深入研究1.tcp三次握手，四次挥手 2.syn洪水攻击 3.服务器高并发情况下会有大量的time_wait状态的优化方法）

- 方法一：
  ```
  #加入一条socket配置，重用ip和端口
  phone=socket(AF_INET,SOCK_STREAM)
  phone.setsockopt(SOL_SOCKET,SO_REUSEADDR,1) #就是它，在bind前加
  phone.bind(('127.0.0.1',8080))
  ```

- 方法二
  ```
  发现系统存在大量TIME_WAIT状态的连接，通过调整linux内核参数解决，
  vi /etc/sysctl.conf
  编辑文件，加入以下内容：
  net.ipv4.tcp_syncookies = 1
  net.ipv4.tcp_tw_reuse = 1
  net.ipv4.tcp_tw_recycle = 1
  net.ipv4.tcp_fin_timeout = 30
  然后执行 /sbin/sysctl -p 让参数生效。
  net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
  net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
  net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
  net.ipv4.tcp_fin_timeout 修改系統默认的 TIMEOUT 时间
  ```

###6.TCP工作流程图

![socket流程图](https://images.cnblogs.com/cnblogs_com/goodcandle/socket3.jpg)

###7.网卡的mtu

- 一般max 1500Btyles
- 分片越多，重组越耗时，网络性能越差
- socket的发送还是接收最好不要超过8k(8096bytes)

###8.部分模块使用

- iter(obj,'b') 迭代obj，只要遇到对象等于b即立刻停止迭代
- re 见 [re博客](http://www.cnblogs.com/yuanchenqi/articles/5732581.html)

###9.socketserver并发

- 基于tcp的套接字，关键就是两个循环，一个链接循环，一个通信循环
- socketserver模块中分两大类：server类（解决链接问题）和request类（解决通信问题）

> **server类**

![](https://images2015.cnblogs.com/blog/1036857/201705/1036857-20170505014200961-1776184607.png)

[](https://images2015.cnblogs.com/blog/1036857/201705/1036857-20170505014200961-1776184607.png)

> **request类**

![](https://images2015.cnblogs.com/blog/1036857/201705/1036857-20170505014309914-771361140.png)

> **继承关系**

![](https://images2015.cnblogs.com/blog/1036857/201705/1036857-20170505015158101-334152905.png)

![](https://images2015.cnblogs.com/blog/1036857/201705/1036857-20170505015247148-219054764.png)

![](https://images2015.cnblogs.com/blog/1036857/201705/1036857-20170505015356492-1711228984.png)



###10.json vs pickle

- json.dumps obj->str
- pickle.dumps  obj->btyles
- [区别可以见博客](http://www.cnblogs.com/yuanchenqi/articles/5732581.html)

###11. socketserver源码分析

- TCP  self.request == conn
- UDP self.request == data tuple

###12.如何引入socket模块

- from socket import *

###13.认证客户端合法性

- 链接合法性
- 通信合法性

> code 见 part31

###14.socket实现ftp

> 要求：

![ftp要求](https://images2015.cnblogs.com/blog/1036857/201703/1036857-20170303202328563-1390338632.png)

- 代码请参考 part32


###15.线程Part33

- 并发
- threading.Thread()模块
- join (位置很重要，防止串行)子线程结束之后才能向下走
- setDaemon(True) start之前设置，主线程退出之后 子线程也退出
- threading.active_count() 几个线程活着
- 调用方式：
  - 继承 重写run方法
  - 直接调用threading

###16.锁

####同步锁

- lock=threading.Lock() **需要考虑在哪加锁**
- lock.acuqire()
- lock.release()
- 缺点：锁是串行的

####死锁 递归锁

- code 见 part34
- 如何解决？
  - 用threading.Rlock() 原理是内置一个计数器,目的是所有线程都抢这一把锁，通过计数器计算

####同步条件Event对象(同步)

- e=threading.Event()
- e.wait() 阻塞，一旦被设定，等同于pass
- e.set() 设定
- e.isSet() 默认false
- e.clear() 清除

####信号量(同步)

- 类似停车位的概念
- 锁的一种
- s=threading.Semaphore()
- s.acquire(num) 同时分配多个锁
- s.release()

####队列 (**重点掌握**)

- 线程安全的,而且是多线程的。相对于列表,list是不安全的
- import queue 线程队列
- q=queue.Queue(num) 先进先出
- q.get(block=False) 默认为True ，如果设置false,则代表如果拿不到数据就报错
- q.put(block=False) 同上 带block=False的put相当于q.nowait()
- queue.LiFoQueue() 先进后出
- queue.PriorityQueue() 设置优先级
- q.qsize() 队列长度
- q.empty() 是否为空
- q.full() 是否满

#####生产者-消费者模型

####栈

####进程

#####多进程模块

- multiprocessing.Process()
- daemon=True
- 调用方式：
  - 直接调用模块
  - 继承模块，重写run方法
- IPC进程间通信机制：
  - 进程队列Queue
  - 管道
  - Managers：数据共享

####并发 vs 并行

#####并发：系统具有处理多个任务(动作)的能力

#####并行：系统具有**<同时>**处理多个任务的能力，多核cpu同时处理多个任务

#####并行是并发的一个子集



####同步 vs 异步

#####同步：当进程执行到一个IO(等待外部数据)-----等待

#####异步：当进程执行到一个IO(等待外部数据)——不等



###17.GIL

####问题？为什么程序开启了多线程效率反而没提高多少？多核没用上？

- 在解释器里保证线程安全—py创始人考虑
- 因为有了GIL，所以同一时刻只有一个线程被cpu执行
- 线程切换消耗了太多时间
- 提高并发方案：
  - 利用进程
  - 多进程+协程

###18.python使用场景

#####对于IO密集型任务：

- python多线程是有意义的


- 多进程+协程

#####对于计算密集型任务：

- 不推荐python多线程，可以采用多进程+协程





















