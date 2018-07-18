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

## 能做什么
### 检查死锁
### CPU使用过高定位
### 线程运行分析