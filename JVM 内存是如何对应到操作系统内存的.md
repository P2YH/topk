### JVM 内存结构

![Java Memory Structure | JVM](https://www.betsol.com/wp-content/uploads/2017/06/JVM-Memory-Model.jpg)

- 堆内存

  - 堆内存是运行时数据区域，java所有的类实例和数组都在堆上分配。

  - 在程序启动时伴随JVM启动创建并且有可能扩容缩容。

  - heap的大小可以用-Xms配置；最大heap大小可以用-Xmx配置，默认最大堆大小是64M。heap可以根据垃圾回收策略修改或者动态调整大小。

- 非堆内存

  Java 虚拟机管理堆之外的内存

  - 伴随JVM启动创建
  - 方法区属于非堆内存
  - 存储每个类结构，例如：运行时常量池、字段和方法数据，以及方法和构造方法的代码。
  - 默认64MB，可以用-XX:MaxPermSize配置

- 其他内存

  存储JVM自己的代码、JVM内部结构体、已加载配置agent代码和数据等。



JVM堆内存结构

![Java Heap Memory Structure | Betsol](https://www.betsol.com/wp-content/uploads/2017/06/java-memory-management-1.jpg)