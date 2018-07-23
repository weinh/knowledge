# Java命令——jstack
jstack是JDK里面自带的工具，可以通过该工具查看Java进程执行情况，显示出具体线程堆栈信息，据了解也可以分析core文件（目前没有使用过），本文讲的都是分析Java进程

## 命令详解
```
jstack.exe --help
Usage:
    jstack [-l] <pid>
    进程还没有挂，使用该命令需要一个pid（Java进程ID）
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
    进程挂了，使用参数强制输出
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
    分析一个core文件
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
    连接到远程主机，输出服务器的进程信息
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    如果Java进程没有响应可以使用该参数，强制输出
    -m  to print both java and native frames (mixed mode)
    混合模式，将输出包括本地方法栈信息
    -l  long listing. Prints additional information about locks
    更详细的清单，附加锁信息
    -h or -help to print this help message
    帮助
```
## jstack输出详解
```

2018-07-23 22:30:02
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.121-b13 mixed mode):

"RMI TCP Connection(2)-169.254.60.191" #17 daemon prio=5 os_prio=0 tid=0x000000001d70e800 nid=0x3bc0 runnable [0x000000001f62e000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(Unknown Source)
	at java.net.SocketInputStream.read(Unknown Source)
	at java.net.SocketInputStream.read(Unknown Source)
	at java.io.BufferedInputStream.fill(Unknown Source)
	at java.io.BufferedInputStream.read(Unknown Source)
	- locked <0x0000000781905cb0> (a java.io.BufferedInputStream)
	at java.io.FilterInputStream.read(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport.handleMessages(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$0(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler$$Lambda$5/1083124264.run(Unknown Source)
	at java.security.AccessController.doPrivileged(Native Method)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- <0x000000078134a6c0> (a java.util.concurrent.ThreadPoolExecutor$Worker)

"JMX server connection timeout 16" #16 daemon prio=5 os_prio=0 tid=0x000000001d737800 nid=0x1a5c in Object.wait() [0x000000001eb2f000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000007815774a8> (a [I)
	at com.sun.jmx.remote.internal.ServerCommunicatorAdmin$Timeout.run(Unknown Source)
	- locked <0x00000007815774a8> (a [I)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- None

"RMI Scheduler(0)" #15 daemon prio=5 os_prio=0 tid=0x000000001d736800 nid=0x8ec waiting on condition [0x000000001ea2e000]
   java.lang.Thread.State: TIMED_WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007812ed7a0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.parkNanos(Unknown Source)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(Unknown Source)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(Unknown Source)
	at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.getTask(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- None

"RMI TCP Connection(1)-169.254.60.191" #14 daemon prio=5 os_prio=0 tid=0x000000001d735800 nid=0x1988 runnable [0x000000001e92d000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(Unknown Source)
	at java.net.SocketInputStream.read(Unknown Source)
	at java.net.SocketInputStream.read(Unknown Source)
	at java.io.BufferedInputStream.fill(Unknown Source)
	at java.io.BufferedInputStream.read(Unknown Source)
	- locked <0x0000000781534bd8> (a java.io.BufferedInputStream)
	at java.io.FilterInputStream.read(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport.handleMessages(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$0(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler$$Lambda$5/1083124264.run(Unknown Source)
	at java.security.AccessController.doPrivileged(Native Method)
	at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- <0x0000000781346a90> (a java.util.concurrent.ThreadPoolExecutor$Worker)

"RMI TCP Accept-0" #13 daemon prio=5 os_prio=0 tid=0x000000001d6b8000 nid=0x359c runnable [0x000000001e72e000]
   java.lang.Thread.State: RUNNABLE
	at java.net.DualStackPlainSocketImpl.accept0(Native Method)
	at java.net.DualStackPlainSocketImpl.socketAccept(Unknown Source)
	at java.net.AbstractPlainSocketImpl.accept(Unknown Source)
	at java.net.PlainSocketImpl.accept(Unknown Source)
	- locked <0x00000007812f7568> (a java.net.SocksSocketImpl)
	at java.net.ServerSocket.implAccept(Unknown Source)
	at java.net.ServerSocket.accept(Unknown Source)
	at sun.management.jmxremote.LocalRMIServerSocketFactory$1.accept(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$AcceptLoop.executeAcceptLoop(Unknown Source)
	at sun.rmi.transport.tcp.TCPTransport$AcceptLoop.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- None

"pool-1-thread-1" #11 prio=5 os_prio=0 tid=0x000000001d882800 nid=0x33dc runnable [0x000000001e22f000]
   java.lang.Thread.State: RUNNABLE
	at jvm.jstack.JstackOut.lambda$threadPool$1(JstackOut.java:33)
	at jvm.jstack.JstackOut$$Lambda$2/135721597.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- <0x00000007810a8a10> (a java.util.concurrent.ThreadPoolExecutor$Worker)

"singleThread" #10 prio=5 os_prio=0 tid=0x000000001d87d000 nid=0x2d40 runnable [0x000000001e12f000]
   java.lang.Thread.State: RUNNABLE
	at jvm.jstack.JstackOut.lambda$singleThread$0(JstackOut.java:24)
	at jvm.jstack.JstackOut$$Lambda$1/834600351.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

   Locked ownable synchronizers:
	- None

"Service Thread" #9 daemon prio=9 os_prio=0 tid=0x000000001c11c000 nid=0x306c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C1 CompilerThread2" #8 daemon prio=9 os_prio=2 tid=0x000000001c0cc800 nid=0x954 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread1" #7 daemon prio=9 os_prio=2 tid=0x000000001c0c9800 nid=0xcdc waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"C2 CompilerThread0" #6 daemon prio=9 os_prio=2 tid=0x000000001c0c3800 nid=0x3b28 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000001c0c2800 nid=0x1f18 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x000000001d573800 nid=0x39cc runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
	- None

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000000050ba800 nid=0x22b0 in Object.wait() [0x000000001d42f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x0000000780d88ec8> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(Unknown Source)
	- locked <0x0000000780d88ec8> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(Unknown Source)
	at java.lang.ref.Finalizer$FinalizerThread.run(Unknown Source)

   Locked ownable synchronizers:
	- None

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x000000001c089000 nid=0x76c in Object.wait() [0x000000001d32f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x0000000780d86b68> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Unknown Source)
	at java.lang.ref.Reference.tryHandlePending(Unknown Source)
	- locked <0x0000000780d86b68> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Unknown Source)

   Locked ownable synchronizers:
	- None

"main" #1 prio=5 os_prio=0 tid=0x0000000004fc0800 nid=0x16c runnable [0x0000000004f3f000]
   java.lang.Thread.State: RUNNABLE
	at jvm.jstack.JstackOut.main(JstackOut.java:19)

   Locked ownable synchronizers:
	- None

"VM Thread" os_prio=2 tid=0x000000001c086000 nid=0x37c8 runnable 

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x0000000004fd6800 nid=0x2c08 runnable 

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x0000000004fd8000 nid=0x26e8 runnable 

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x0000000004fda000 nid=0x930 runnable 

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x0000000004fdb800 nid=0x2f48 runnable 

"VM Periodic Task Thread" os_prio=2 tid=0x000000001d5ee800 nid=0x343c waiting on condition 

JNI global references: 356
```
## 能做什么
### 检查死锁
### CPU使用过高定位
### 线程运行分析