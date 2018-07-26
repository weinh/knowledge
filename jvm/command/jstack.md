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
```java
public class JstackOut {

    public static void main(String[] args) {
        singleThread();
        threadPool();
        while (true) ;
    }

    private static void singleThread() {
        Thread singleThread = new Thread(() -> {
            while (true) ;
        });
        singleThread.setName("singleThread");
        singleThread.start();
    }

    private static void threadPool() {
        Executor executor = Executors.newFixedThreadPool(4);
        executor.execute(() -> {
            Thread.currentThread().setName("thread pool 1");
            while (true) ;
        });
        executor.execute(() -> {
            Thread.currentThread().setName("thread pool 2");
        });
    }
}
```
运行后获取线程堆栈信息
```
[root@hsipcc opt]# jstack 12175
2018-07-26 15:32:49
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.144-b01 mixed mode):

"Attach Listener" #9 daemon prio=9 os_prio=0 tid=0x00007f0fb8001000 nid=0x3849 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #8 daemon prio=9 os_prio=0 tid=0x00007f0ff40c6800 nid=0x3834 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #7 daemon prio=9 os_prio=0 tid=0x00007f0ff40bb800 nid=0x3833 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007f0ff40b9800 nid=0x3832 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f0ff40b6800 nid=0x3831 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f0ff40b5800 nid=0x3830 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f0ff4082800 nid=0x382f in Object.wait() [0x00007f0fdec6e000]
   java.lang.Thread.State: WAITING (on object monitor)
    at java.lang.Object.wait(Native Method)
    - waiting on <0x000000076ce08ec8> (a java.lang.ref.ReferenceQueue$Lock)
    at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
    - locked <0x000000076ce08ec8> (a java.lang.ref.ReferenceQueue$Lock)
    at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
    at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f0ff407e000 nid=0x382e in Object.wait() [0x00007f0fded6f000]
   java.lang.Thread.State: WAITING (on object monitor)
    at java.lang.Object.wait(Native Method)
    - waiting on <0x000000076ce06b68> (a java.lang.ref.Reference$Lock)
    at java.lang.Object.wait(Object.java:502)
    at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
    - locked <0x000000076ce06b68> (a java.lang.ref.Reference$Lock)
    at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"main" #1 prio=5 os_prio=0 tid=0x00007f0ff4008800 nid=0x3828 runnable [0x00007f0ffa047000]
   java.lang.Thread.State: RUNNABLE
    at jvm.jstack.JstackOut.main(JstackOut.java:25)

"VM Thread" os_prio=0 tid=0x00007f0ff4076000 nid=0x382d runnable 

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00007f0ff401d800 nid=0x3829 runnable 

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00007f0ff401f000 nid=0x382a runnable 

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00007f0ff4021000 nid=0x382b runnable 

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00007f0ff4022800 nid=0x382c runnable 

"VM Periodic Task Thread" os_prio=0 tid=0x00007f0ff40c9800 nid=0x3835 waiting on condition 

JNI global references: 6
```
其中
```
"main" #1 prio=5 os_prio=0 tid=0x00007f0ff4008800 nid=0x3828 runnable [0x00007f0ffa047000]
   java.lang.Thread.State: RUNNABLE
    at jvm.jstack.JstackOut.main(JstackOut.java:25)
```
就是一个线程堆栈信息

`"main"`：线程名称，如果使用线程池的话，一般要求自定义一个线程名，以便排查问题

tid：Java Thread id

nid：native线程的id，他可以通过linux命令：`ps -mp pid -o THREAD,tid,time`可以找到一个Java进程所有的线程，包含CPU使用情况，线程ID，将命令得到的线程ID从十进制转换成十六进制，可以得到nid

java.lang.Thread.State：线程状态，初始`NEW`运行`RUNNABLE`阻塞`BLOCKED`等待`WAITING`超时等待`TIME_WAITING`终止`TERMINATED`
## 能做什么
### 检查死锁
### CPU使用过高定位
### 线程运行分析