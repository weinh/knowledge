# Lambda
自从使用了JDK1.8很多地方用到了Lambda表达式，短短几行代码就实现了需要的效果，下面介绍下实现原理
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
Java