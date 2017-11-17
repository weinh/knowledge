# 基础语法
## 基本类型
* byte
* short
* int
* long
* float
* double
* boolean
* char

每个基本类型都有对应的封装类

1. 基本类型只能按值传递，对象按引用传递
2. 从性能上基本类型在堆栈上创建，对象类型在堆上创建，对象应用在堆栈上创建
> 堆栈上创建有可能引起内存泄露，Integer i = new Integer(10);new XXX在堆上创建，Integer i在堆栈上创建

## String
字符串对象分两种，一个是常量池里面的字符串，一个是堆上创建的字符串

匹配相等，一般有两种选择equals或==，这两种方式是有区别的，equals是表示值是否相等，==是表示是否是同一个对象

==的成立与否在编译器就已经确定了，比如
```java
String ok="ok";
String ok1=new String("ok");
System.out.println(ok==ok1);//fasle

String ok="apple1";
String ok1="apple"+1;
System.out.println(ok==ok1);//true

String ok="apple1";
int temp=1;
String ok1="apple"+temp;
System.out.println(ok==ok1);//false

String ok="apple1";
final int temp=1;
String ok1="apple"+temp;
System.out.println(ok==ok1);//true


public static void main(String[] args) {
    String ok="apple1";
    final int temp=getTemp();
    String ok1="apple"+temp;
    System.out.println(ok==ok1);//false
}
public static int getTemp()
{
  return 1;
}
```

intern()方法的作用是常量池的扩展，当调用该方法时会检查常量池有没有该常量，没有的话创建一个，并返回其引用

String具有不可变性，当String变量需要经常变换时，会产生很多变量，应考虑StringBuffer或StringBuilder提高效率
## StringBuffer & StringBuilder
他们具有功能的父类AbstractStringBuilder，处理了大部分工作，将字符串附加到char[] value中，从而提高效率

其中StringBuffer是线程安全的，因为他在方法上加了synchronized的修饰