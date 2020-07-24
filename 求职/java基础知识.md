### JDK,JRE
​	·JDK:Java Development Kit 的简称，Java 开发工具包，提供了 Java 的开发环境和运行环境。
​	·JRE:Java Runtime Environment 的简称，Java 运行环境，为 Java 的运行提供了所需环境。

### equals比较的是值，==比较引用地址。

### java数据类型
- String 不属于基础类型，基础类型有 8 种：byte、boolean、char、short、int、float、long、double，而 String 属于对象。
  	https://blog.csdn.net/weixin_36431280/article/details/78430786

### String str="i"与 String str=new String("i")一样吗？
- 不一样，因为内存的分配方式不一样。String str="i"的方式，Java 虚拟机会将其分配到常量池中；而 String str=new String("i") 则会被分到堆内存中。

### java的跨平台原理
- 不同的操作系统底层操作集不同，因此安装适配不同系统的java虚拟机，从而对外提供统一的接口

### java的面向对象是什么意思？
- 面向对象四大特征：封装、抽象、继承、多态。
  - 封装：将对象封装成比较自治和封闭的个体。
  - 抽象：有选择的总结现实中的对象的属性，将其抽象为类。
  - 继承：定义新的类时，可以在已有类的基础上，进行属性的继承和修改。
  - 多态：一个方法在父类和子类中有不同的实现，比如方法覆写。

