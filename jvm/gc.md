# 垃圾回收
为什么需要垃圾回收呢？

Java语言提供了动态分配与自动回收内存，一切都那么的自动化了

可是实际项目中经常会发生内存溢出，内存泄漏，当垃圾回收成为系统达到更高并发量的瓶颈时，那么我们就要深入学习研究垃圾回收的原理了

内存模型中，`程序计数器`，`本地方法栈`，`虚拟机栈`是随着线程的创建而创建，结束而消亡的，所以内存回收不太考虑这三块区域

而`方法区`，`堆内存`实在运行时才能确定内存大小，具体的空间都是动态分配的，回收也是自动的，所以接下来垃圾回收说的就是这两块区域
## 那些对象已死
Java堆中几乎存放这所有的对象实例，垃圾回收前需要确定那些实例还`活着`，那些实例已经`死了`（不会被任何途径使用）
### 引用计数法
其实就是，其实就是有个引用计数器，每引用一次加1，引用失效减1，可是这种无法处理循环引用的情况

写个例子测试下
```java
public class ReferenceCountingGC {
    private byte b[] = new byte[1024 * 1024 * 2];
    private ReferenceCountingGC instance;

    /**
     * 运行时，加入jvm参数查看GC情况：-XX:+PrintGCDetails
     *
     * @param args 参数
     */
    public static void main(String[] args) {
        ReferenceCountingGC refA = new ReferenceCountingGC();
        ReferenceCountingGC refB = new ReferenceCountingGC();
        refA.instance = refB;
        refB.instance = refA;

        refA = null;
        refB = null;
        System.gc();
    }
}
```
执行该方法，配置上JVM参数-XX:+PrintGCDetails，可以看到GC的日志打印记录，如下
```
[GC (System.gc()) [PSYoungGen: 7429K->792K(38400K)] 7429K->800K(125952K), 0.0011108 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 792K->0K(38400K)] [ParOldGen: 8K->660K(87552K)] 800K->660K(125952K), [Metaspace: 3480K->3480K(1056768K)], 0.0067504 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 38400K, used 333K [0x00000000d5f80000, 0x00000000d8a00000, 0x0000000100000000)
  eden space 33280K, 1% used [0x00000000d5f80000,0x00000000d5fd34a8,0x00000000d8000000)
  from space 5120K, 0% used [0x00000000d8000000,0x00000000d8000000,0x00000000d8500000)
  to   space 5120K, 0% used [0x00000000d8500000,0x00000000d8500000,0x00000000d8a00000)
 ParOldGen       total 87552K, used 660K [0x0000000081e00000, 0x0000000087380000, 0x00000000d5f80000)
  object space 87552K, 0% used [0x0000000081e00000,0x0000000081ea5218,0x0000000087380000)
 Metaspace       used 3487K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
```
从日志上面看到`[PSYoungGen: 7429K->792K(38400K)]`，说明并没有因为两个对象相互引用而没有回收他们
### 可达性径分析算法
该算法的基本思路就是通过一系列称为`GC Roots`的对象作为起始点，从这些节点开始向下搜索，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的

可作为GC Roots的对象包括下面几种
* 虚拟机栈（帧栈中的本地变量表）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈JNI（本地方法）引用的对象
### 是死是活
是不是可达性分析认为不可用后对象一定会被回收呢？如果对象进行可达性分析后发现不可用，那么将会进行一次标记并进行一次筛选，进入`缓刑`阶段，筛选的条件是该对象是否有必要执行finalize()方法

当对象没有覆盖finalize()方法或finalize()方法已经被虚拟机调用过，虚拟机将这两种情况视为没有必要执行

当第一次被标记后，有机会通过执行finalize()方法`拯救`自己，看下示例代码说明下
```java
public class FinalizeGC {
    private static FinalizeGC SAVA_HOOK = null;

    public void isAlive() {
        System.out.println("我还活着！");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize方法执行，逃过一劫");
        FinalizeGC.SAVA_HOOK = this;
    }

    public static void main(String[] args) throws InterruptedException {
        SAVA_HOOK = new FinalizeGC();

        SAVA_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVA_HOOK != null) {
            SAVA_HOOK.isAlive();
        } else {
            System.out.println("终于死了！");
        }

        SAVA_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVA_HOOK != null) {
            SAVA_HOOK.isAlive();
        } else {
            System.out.println("终于死了！");
        }
    }
}
```
执行结果
```
finalize方法执行，逃过一劫
我还活着！
终于死了！
```
从执行结果来看，finalize()方法只会被执行一次，第一次执行后该对象可以从回收的边缘拉回来，可是没办法逃出毁灭的命运

finalize()方法不建议大家使用，运行代价高，不确定性大，finalize()能做的工作使用try-finally或者其他方式都可以做得更好，更及时
### 回收方法区
Java虚拟机规范中说过可以不要求虚拟机在方法区实现垃圾收集，而且在方法区中进行垃圾手机的性价比一般比较低

永久代的垃圾收集主要回收两部分内容：废气常量和无用的类，回收废弃常量和回收Java堆中的对象非常类似

不过要判断一个类是否是无用的类条件比较苛刻
* 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例
* 加载该类的ClassLoader已经被回收
* 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

满足以上三个条件，只是说"可以"被回收而不是必然会回收，是否回收HotSpot虚拟机提供了一些参数进行控制：`-Xnoclassgc`，也可以通过一些参数查看类加载卸载信息