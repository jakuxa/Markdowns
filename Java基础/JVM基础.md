## JVM基础知识结构

![image-20200707103248970](http://www.jakuxa.cn:11020/images/2020/07/24/image-20200707103248970.png)

### JVM调优

> 99%的调优是在**方法区**和**堆**中调优，而其中**堆**的调优又占更大的比重



### 类加载器

> 有四种加载器
>
> 1、虚拟机自带的加载器
>
> 2、APP：应用程序(系统类)加载器
>
> 3、EXT：拓展类加载器
>
> 4、BOOT：启动类(根)加载器

根加载器无法通过getClassLoader()方法获得，因为它是用C写的，其他则是java写的，便可以获得

![image-20200707104644361](http://www.jakuxa.cn:11020/images/2020/07/24/image-20200707104644361.png)

```java
//类是模板，对象才是实例

Car car1 = new Car();
Car car2 = new Car();
Car car3 = new Car();

Class<? extends Car> aClass1 = car1.getClass();
Class<? extends Car> aClass2 = car2.getClass();
Class<? extends Car> aClass3 = car3.getClass();

sout(aClass1.getClass().getClassLoader())

//aClass1和aClass2和aClass3都是一样的，都是Car.class模板
```



### 双亲委派机制

> 该机制是安全导向
>
> 1、类加载器收到类加载的请求
>
> 2、将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器
>
> 3、启动类加载器检查是否能够加载请求的这个类，能加载就结束，不能加载就抛出异常，通知下一层加载器去尝试加载。
>
> 4、重复步骤3

- APP--->EXT--->BOOT(最终执行）



### 沙箱安全机制

> 限制java运行的环境，将方法限制在jvm的环境内。

![image-20200707112938351](http://www.jakuxa.cn:11020/images/2020/07/24/image-20200707112938351.png)

详见：https://blog.csdn.net/qq_30336433/article/details/83268945



### 本地方法接口（JNI）

> java native interface，Java达不到的，使用底层C实现的方法接口

```java
//携带native修饰词的，会进入本地方法栈
```





### PC寄存器



### 方法区





### 方法栈





### **堆**

```java
//堆内存调优，首先尝试扩大堆内存
//Xms初始化内存，Xmx总内存
-Xms1024m -Xmx1024m -XX:+PrintGCDetails
```

```java
//利用dump生成诊断内存文件，使用jprofile打开
-Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError
```



### GC

