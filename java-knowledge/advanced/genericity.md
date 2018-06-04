# 泛型
泛型编程是一种通过参数化的方式将数据处理与数据类型解耦的技术，通过对数据类型施加约束（比如Java中的有界类型）来保证数据处理的正确性，又称为参数类型或者参数对态性

泛型最著名的应用就是容器，Java的集合框架
## 泛型的实现方式
Java的泛型是伪泛型，编译为字节码时参数类型会在代码中被擦除，单独记录在Class文件的attributes域中，而在使用泛型处做类型检查和类型转换，假设参数类型的占位符为T，擦除规则如下：
* `<T>`擦除后变为Object
* `<? extends A>`擦除后变为A
* `<? super A>`擦除后变为Object
* `<X,Y>`擦除后变为同一父类最小级

上述擦除规则叫做`保留上界`
## 类型擦除引起的问题及解决方法
* 先检查，再编译，以及检查编译的对象和引用传递问题
```
List fruits0 = new ArrayList();
List<Fruit> fruits1 = new ArrayList<>();
List<Fruit> fruits3 = new ArrayList();
List fruits2 = new ArrayList<>();

List<Apple> apples = new ArrayList<>();
apples.add(new Apple());
apples.add(new RedApple());
// 错误，保留上界，检查不通过
// apples.add(new Banana());
// 错误
// 从下面的例子看，很明显，objects1可以是任意类型，做了赋值引用后，strings1必定会出现类型转换错误，所以避免了这种情况
// ArrayList<String> list1 = new ArrayList<Object>();
// ArrayList<Object> objects1 = new ArrayList<>();
// objects1.add("0");
// objects1.add(0);
// ArrayList<String> strings1 = objects1;
//
// 从下面的例子看，有两个问题1、本身泛型就算为了方便使用，而不需要强制转换2、如果objects1操作add，到时候取出来的是String还是Object呢
// ArrayList<Object> list2 = new ArrayList<String>();
// ArrayList<String> strings1 = new ArrayList<>();
// strings1.add("1");
// strings1.add("2");
// ArrayList<Object> objects1 = strings1;
```
* 自动强制转换
既然类型被擦除了，那么为什么可以直接使用呢
ArrayList中的方法，帮我们强制转换了，有一些直接赋值的字节码帮我们自动转换了
```
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```
* 类型擦除与多态的冲突和解决办法
```
class Plate<T> {
    private T value;

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}

class FruitPlate extends Plate<Fruit> {
    @Override
    public Fruit getValue() {
        return super.getValue();
    }

    @Override
    public void setValue(Fruit value) {
        super.setValue(value);
    }
}

public void polymorphism() {
    FruitPlate fruitPlate = new FruitPlate();
    fruitPlate.setValue(new Fruit());
    // 错误，证明这个不是重载，是重写
    // fruitPlate.setValue(new Object());
}
```
怎么做到重写，而不是重载呢，那就是`桥方法`，看下字节码
```
javap -c Genericity$FruitPlate.class
Compiled from "Genericity.java"
class java_knowledge.advanced.Genericity$FruitPlate extends java_knowledge.advanced.Genericity$Plate<java_knowledge.advanced.Genericity$Fruit> {
  final java_knowledge.advanced.Genericity this$0;

  java_knowledge.advanced.Genericity$FruitPlate(java_knowledge.advanced.Genericity);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #1                  // Field this$0:Ljava_knowledge/advanced/Genericity;
       5: aload_0
       6: aload_1
       7: invokespecial #2                  // Method java_knowledge/advanced/Genericity$Plate."<init>":(Ljava_knowledge/advanced/Genericity;)V
      10: return

  public java_knowledge.advanced.Genericity$Fruit getValue();// 编译public Fruit getValue()方法
    Code:
       0: aload_0
       1: invokespecial #3                  // Method java_knowledge/advanced/Genericity$Plate.getValue:()Ljava/lang/Object;
       4: checkcast     #4                  // class java_knowledge/advanced/Genericity$Fruit
       7: areturn

  public void setValue(java_knowledge.advanced.Genericity$Fruit);// 编译public void setValue(Fruit value)方法
    Code:
       0: aload_0
       1: aload_1
       2: invokespecial #5                  // Method java_knowledge/advanced/Genericity$Plate.setValue:(Ljava/lang/Object;)V
       5: return

  public void setValue(java.lang.Object); // JVM生成的桥方法
    Code:
       0: aload_0
       1: aload_1
       2: checkcast     #4                  // class java_knowledge/advanced/Genericity$Fruit先检查
       5: invokevirtual #6                  // Method setValue:(Ljava_knowledge/advanced/Genericity$Fruit;)V再调用public void setValue(Fruit value)方法
       8: return

  public java.lang.Object getValue();
    Code:
       0: aload_0
       1: invokevirtual #7                  // Method getValue:()Ljava_knowledge/advanced/Genericity$Fruit;
       4: areturn
}
```
正常情况下public java.lang.Object getValue();和public java_knowledge.advanced.Genericity$Fruit getValue();不可能同时出现，但是虚拟机为了实现泛型自己开了个后门
* 泛型类型变量不能是基本数据类型
## 通配符和T的区别
* T

    作用于模板上，用于将数据类型进行参数化，不能用于实例化对象
    
* ?

    在实例化对象的时候，不确定泛型参数的具体类型时，可以使用通配符进行对象定义
```
<T> 等同于 <T extends Object>
<?> 等同于 <? extends Object>
```
例子1，key-value数据类型进行<K, V>参数化，而不可以使用通配符：
```java
class Container<K, V> {
    private K key;
    private V value;

    public Container(K key, V value) {
        this.key = key;
        this.value = value;
    }
}
```
例子2，当不确定集合存储的类型，可以使用List<? extends Fruit>
```
List<? extends Fruit> list = null;
list = new ArrayList<Apple>();
list = new ArrayList<Banana>();
```
## 上界类型通配符<? extends T>
```
List<? extends Fruit> list = null;
list = new ArrayList<Apple>();
// 正确
Fruit fruit = list.get(0);
list = new ArrayList<Banana>();
// 错误
Banana banana = list.get(0);
// 错误
list.add(new Banana());
```
总结：上界类型通配符add方法受限，但可以获取列表中的各种类型的数据，并赋值给父类型的引用，因此如果你想从一个数据类型获取数据，使用该通配符，限定通配符总是包括自己
## 下界类型通配符<? super T>
```
List<? super Apple> list = null;
list = new ArrayList<Fruit>();
// 正确
list.add(new Apple());
// 正确
list.add(new RedApple());
// 错误
// Fruit fruit = list.get(0);
// 错误
// Apple apple = list.get(0);
```
总结，下界类型通配符get方法受限，但可以往列表中添加各种数据类型的对象，因此如果你想把对象写入一个数据结构里，使用次通配符，限定通配符总是包括自己
## 总结
* 限定通配符总是包括自己
* 上界类型通配符：add方法受限
* 下界类型通配符：get方法受限
* 如果你想从一个数据结构里获取数据，使用<? extends T>通配符
* 如果你想把对象写入一个数据结构里，使用<? super T>通配符
* 如果你既想存，又想取，那就别用通配符
* 不能同时声明泛型通配符上界和下界