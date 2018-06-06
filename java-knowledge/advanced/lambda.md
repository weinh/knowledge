# Lambda
以前学过面向过程编程，面向对象编程，其实还有一个面向函数编程也就是函数式编程，lambda表达式就是Java的函数式编程
## 语法
```
List<String> list = new ArrayList<>();
// 单条表达式
list.forEach(s -> System.out.println(s));
// 方法引用
list.forEach(System.out::println);
// 多条表达式
list.forEach(s -> {
    System.out.println(s);
});
// 无参
new Thread(() -> System.out.println(1));
```
从上面的语句来看，lambda表达式可以使代码变得更紧凑，还能够修改方法的能力，简单来说函数的可以接受以函数为单元的入参
## 函数式接口
函数式接口是JDK8为了支持lambda表达式产生的

函数式接口有两个特征：一是接口，二是具有唯一的抽象方法
1. 有且仅有一个抽象方法
2. 可以定义public static方法，默认方法，重写的equals方法，这些都不是抽象方法
3. `@FunctionalInterface`注解不是必须的，如果是编程式接口加不加都没关系，如果不是编程式接口加了该注解编译器会提示报错
```java
@FunctionalInterface
public interface MyFunction {
    default void defaultFunction() {
    }

    static void staticFunction() {
    }

    @Override
    boolean equals(Object object);


    boolean only(String value);
}
```
## 方法引用
有时，我们需要执行的代码再某些类中已经存在，这时就没有必要再去写lambda表达式，可以直接使用方法，这种情况我们称之为方法引用
```
// 一般写法
list.forEach(s -> System.out.println(s));
// 方法引用
list.forEach(System.out::println);
```
构造方法引用
```
Class::new  相当于  String str = new String();
Class[]::new    相当于  String[] str = new String[1];
```
## 作用域
lambda表达式的变量作用域和内部类非常相似，条件相对以前放宽了，以前内部类想要引用外部类的变量必须是final修饰的，现在不需要了只要只读不修改就可以
```
String scope = "";
new Thread(() -> System.out.println(scope));
```
lambda表达式内定义的变量和外部类中的变量作用域相同，所以外部定义了，表达式就不能重复定义了，lambda表达式使用this关键字，指向的是外部类
```
public void testThis() throws InterruptedException {
    String param = "";
    Thread thread = new Thread(() -> {
        // 错误 不能重复定义
        // String param = "";
        System.out.println(this);
    });
    thread.start();
}
```
执行结果
```
java_knowledge.advanced.Lambda@62b7b91
```
## 实现原理
简单了解了下，lambda表达式，下面来看JDK8是如何实现的呢，看个例子
```java
@FunctionalInterface
interface MyFunction {
    default void defaultFunction() {
    }

    static void staticFunction() {
    }

    @Override
    boolean equals(Object object);

    void only(String value);
}

public class TestFunction {
    public static void println(String string, MyFunction myFunction) {
        myFunction.only(string);
    }

    public static void main(String[] args) {
        println("test", (x) -> System.out.println(x));
    }
}
```
我们来看下编译后的代码
```
javap.exe -p TestFunction.class
Compiled from "TestFunction.java"
public class java_knowledge.advanced.TestFunction {
  public java_knowledge.advanced.TestFunction();
  public static void println(java.lang.String, java_knowledge.advanced.MyFunction);
  public static void main(java.lang.String[]);
  private static void lambda$main$0(java.lang.String);
}
```
从代码上来看多了一个private static void lambda$main$0(java.lang.String);方法

如果我们再代码中也创建这个方法
```
private static void lambda$main$0(String string) {

}
```
编译是提示，存在了两个lambda$main$0方法
```
Error:(33, 25) java: 符号lambda$main$0(java.lang.String)与java_knowledge.advanced.TestFunction中的 compiler-synthesized 符号冲突
Error:(1, 1) java: 符号lambda$main$0(java.lang.String)与java_knowledge.advanced.TestFunction中的 compiler-synthesized 符号冲突
```
从上面可以看到，lambda表达式再JDK8中首先会生成一个私有的静态方法，这个方法里面干的事情就是表达式里面的代码，那么如何执行这个私有方法呢

通过调试可以看到执行过程中将会调用LambdaMetafactory的metafactory方法，生成一个内部类
```
public static CallSite metafactory(MethodHandles.Lookup caller,
                                   String invokedName,
                                   MethodType invokedType,
                                   MethodType samMethodType,
                                   MethodHandle implMethod,
                                   MethodType instantiatedMethodType)
        throws LambdaConversionException {
    AbstractValidatingLambdaMetafactory mf;
    mf = new InnerClassLambdaMetafactory(caller, invokedType,
                                         invokedName, samMethodType,
                                         implMethod, instantiatedMethodType,
                                         false, EMPTY_CLASS_ARRAY, EMPTY_MT_ARRAY);
    mf.validateMetafactoryArgs();
    return mf.buildCallSite();
}
```
验证是否真的生成了内部类，可以加`-Djdk.internal.lambda.dumpProxyClasses`参数查看生成的内容

得到文件TestFunction$$Lambda$1.class，反编译看下内容
```
javap.exe -c TestFunction$$Lambda$1.class
final class java_knowledge.advanced.TestFunction$$Lambda$1 implements java_knowledge.advanced.MyFunction {
  public void only(java.lang.String);
    Code:
       0: aload_1
       1: invokestatic  #18                 // Method java_knowledge/advanced/TestFunction.lambda$main$0:(Ljava/lang/String;)V
       4: return
}
```
代码中看到，该内部类的only方法实现，调用了lambda$main$0私有静态方法，最终lambda表达式生成的代码就类似
```java
public class TestFunction {
    public static void println(String string, MyFunction myFunction) {
        myFunction.only(string);
    }

    public static void main(String[] args) {
        String string = "test";
        println(string, new TestFunction().new $Lambda$1());
    }

    /**
     * 编译器生成的私有静态方法
     *
     * @param string 参数
     */
    private static void lambda$main$0(String string) {
        System.out.println(string);
    }

    /**
     * 执行生成的私有静态方法，创建的内部类
     */
    final class $Lambda$1 implements MyFunction {
        @Override
        public void only(String string) {
            lambda$main$0(string);
        }
    }
}
```