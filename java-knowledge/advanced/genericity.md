# 泛型
泛型在编码过程中经常使用，本文将针对通配符`?`相关内容进行讲解
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