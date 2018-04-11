# 类加载器
通过自定义的程序通过全限定名获取描述此类的二进制字节流实现加载的模块

类加载器在类的层次划分，OSGi，热部署，代码加密等领域大放异彩

类的唯一性确定，需要是同一个加载器加载的类才有意义，也就是说类+类加载器决定了唯一性

## 双亲委派模型
![](../resources/image/loader-class.jpg "类加载器")

类加载器分为两种，一种是启动类加载器由C++实现是虚拟机的一部分，另一种是其他类加载器由Java实现独立于虚拟机

系统提供了三种类加载器
1. 启动类加载器（Bootstrap ClassLoader）
> 负责加载${JAVA_HOME}/lib目录下的文件，或者通过参数-Xbootclasspath指定路径
2. 扩展类加载器（Extension ClassLoader）
> 由sun.misc.Launcher$ExtClassLoaders实现，负责加载${JAVA_HOME}/lib/ext目录文件或由系统变量java.ext.dirs指定的目录
3. 应用程序类加载器（Application ClassLoader）
> 由sun.misc.Launcher$AppClassLoader实现，该类加载器是ClassLoader.getSystemClassLoader()的返回值，所以也叫做系统类加载器，负责加载用户类路径（classpath）上的文件

双亲委派模型最明显的优点就是加载类具备优先级概念，比如加载java.lang.Object肯定是由启动类加载器加载的，如果不是使用双亲委派模型，我们自定义一个java.lang.Object然后加载上来就出现多个Object了

### 类加载器例子
jvm.LoaderClass
双亲委派模型
```java
    private static void showLoaderClass() {
        ClassLoader a = LoaderClass.class.getClassLoader();
        do {
            System.out.println(a);
        } while ((a = a.getParent()) != null);
    }
```
输出结果
```
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@1540e19d
```
自定义类加载器
```java
    private static void loadClass() throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Object object = new MyClassLoader().loadClass("jvm.LoaderClass").newInstance();
        System.out.println(object.getClass());
        System.out.println(object.getClass().getClassLoader());
        System.out.println(object instanceof LoaderClass);
    }

    static class MyClassLoader extends ClassLoader {
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            try {
                String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                InputStream is = getClass().getResourceAsStream(fileName);
                if (is == null) {
                    return super.loadClass(name);
                }
                byte[] b = new byte[is.available()];
                is.read(b);
                return defineClass(name, b, 0, b.length);
            } catch (IOException e) {
                e.printStackTrace();
                throw new ClassNotFoundException(name);
            }
        }
    }
```
通过自定义类加载器加载类的结果
```
class jvm.LoaderClass
jvm.LoaderClass$MyClassLoader@7f31245a
false
```