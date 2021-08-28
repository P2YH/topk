### 前言

想要弄懂Java垃圾回收，理解JVM内存模型，Java内存管理是非常重要的。今天我们就来看看Java内存管理，Jvm内存的构成以及如果监控和垃圾回收调优。



### Java（JVM）内存模型

![Java Memory Model, JVM Memory Model, Memory Management in Java, Java Memory Management](https://cdn.journaldev.com/wp-content/uploads/2014/05/Java-Memory-Model-450x186.png)

正如上图所示，JVM内存是分成几部分的。在上层，JVM堆内存是分为年轻代和年老代。

### 内存管理-年轻代（Young Generation）

所有的新objects在年轻代上创建。当年轻代被填满后，垃圾回收启动。这个垃圾回收叫**Minor GC**。年轻代分为三部分：Eden内存，两个S0，S1 Survivor内存。

年轻代的重点：

- 大部分新创建的objects是位于Eden内存空间。
- 当Eden空间被objects占满后，Minor GC启动，所有的幸存objects被移动到其中一个survivor空间。
- Minor GC也负责检查幸存的objects和移动他们到另一个survivor空间。所以在某个时刻，其中一个survivor空间永远是空的。
- 经过多轮垃圾回收后还幸存的Objects，将被移动到老年代内存空间。通常配置一个年龄阈值来晋升年轻代的objects到老年代。

### 内存管理-老年代（Old Generation）

老年代包含的objects是long-lived和在几轮Minor GC后的幸存者。通常，内存满的时候垃圾回收是在老年代内存上进行的。老年代垃圾回收被称为**Major GC**。通常回收需要花费很长时间。

### Stop the World事件

所有的垃圾回收都是“Stop the World”事件，因为所有的应用线程在回收完成前都是stoppde的。

因为年轻代持有的是short-lived objects。所有Minor GC是非常快，快到应用都不受其影响。

然而，Major GC因为要检查所有的live objects所有需要花费很长的时间。Major GC应该降低到最小，因为它将使得你的应用在垃圾回收期间无响应。所以，如果你有一个实时响应的应用并且有许多的Major GC发生，那么你将会有超时错误提醒。

GC回收用时依赖于GC的策略。这也就是为什么我们需要监控和调优GC，来避免高实时响应应用的超时问题。

### Java内存模型-永久代（Permanent Generation）

永久代（Permanent Generation or “Perm Gen” ）持有的是JVM用来描述类和方法的应用的元数据。需要注意的是：永久代不是Java Heap内存的部分。

永久代是被JVM运行时基于被应用使用的类所占用的。永久代也持有Java SE的库类和方法。永久代objects由Full GC负责垃圾回收。

### Java内存模型-方法区（Method Area）

方法区是永久代的一部分空间，用来存储类结构（运行时常量和静态变量）和方法代码和构造方法。

### Java内存模型-内存池（Memory Pool）

内存池是被 JVM内存管理创建的，用来创建不可变（immutable）objects池，如果实现支持的话。String池就是一个这类内存池的很好例子。内存池可以属于Heap或者永久代，这取决于JVM内存管理的实现。

### Java内存模型-运行时常量池（Runtime Constant Pool）

运行时常量池是每个类运行时在类内描述常量池。它持有运行时常量和静态方法。运行时常量池是方法区的一部分。

### Java内存模型-Java栈内存

Java栈内存是被线程执行的使用的。它们包括方法特定变量：short-lived和引用其它objects在堆中被方法引用的。

### 内存管理在Java-Java堆内存交换

Java提供许多内存交换方法，我们可以用来设置内存大小和比率。一些常用的方法如下：

| VM Switch         | VM Switch Description                                        |
| :---------------- | :----------------------------------------------------------- |
| -Xms              | 设置JVM启动时的初始heap大小。                                |
| -Xmx              | 设置最大的heap大小。                                         |
| -Xmn              | 设置年轻代的大小，剩余的空间就是留给老年代的。               |
| -XX:PermGen       | 设置永久代内存的初始大小。                                   |
| -XX:MaxPermGen    | 设置永久代最大Size。                                         |
| -XX:SurvivorRatio | 设置Eden和Survivor空间的比例。例如：年轻代大小是10m，VM切换 -XX:SurvivorRatio=2，那么Eden区是5m，每一个Survivor区是2.5m。默认值是8. |
| -XX:NewRatio      | 设置新/老代的比例。默认是2.                                  |

大多数时间，上面的配置项是绰绰有余的。但是如果你想要了解其他的配置，可以看看[JVM Options Official Page](https://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)

### 内存管理在Java-Java垃圾回收

垃圾回收是负责定义和删除内存中不使用的objects，释放空间以便于未来的进程创建objects时申请。其中一个最好的特性是Java编程语言的 **自动垃圾回收（automatic garbage collection）**，不像其他的编程语言例如C语言需要手动处理申请和释放的工作。

垃圾回收是后台执行的程序负责观察（looks into）所有内存中的objects，并且找出那些不再被任何程序引用的objects。所有的未被引用的objects将被删除和回收空间给出其他objects申请使用。

其中基本的垃圾回收方式包括三个步骤：

1. **标记（Marking）**：第一步负责标记那些objects是在使用中的那些事不再使用。
2. **正常删除（Normal Deletion）**：垃圾回收删除不使用的objects，回收空间给其他objects申请
3. **压缩删除（**Deletion with Compacting**）**：为了更好的性能，在删除不使用的objects后，所有的survived objects可以被移动到一起。这样可以提升新objects申请内存的效率。

这里存在两个问题关于简单的标记和删除过程。

1. 第一点毫无疑问大部分新创建的objects将变成为使用的。
2. 第二点那些经过多轮垃圾回收还是in-use的objects是最有可能在未来回收也变成in-use。

上述简单处理的缺点正是Java垃圾回收分代际的原因，因此有**年轻代**和**老年代**堆内存空间。文中已经解释过ojects是如何在Minor GC和Major GC中扫描和从一个代移动到另外一个代的。

### 内存管理在Java-Java垃圾回收类型

有五种垃圾回收类型可以供我们使用。我们只需要使用JVM开关为应用开启垃圾回收策略。让我们一个一个的看。

1. **串行Serial GC (-XX:+UseSeralGC)**：Serial GC使用简单的**mark-sweep-compact**处理年轻和老年代垃圾回收也就是Minor和Marjor GC。Serial GC是在客户端机器例如我们简单的独立应用和小CPU的机器。它对运行在低内存空间的小型应用是很好用的。

2. **并发Parallel GC (-XX:+UseParallelGC)**: Parallel GC和Serial GC一样，除了它为Minor GC创建N个线程，N是操作系统的CPU核心数。我们可以用JVM配置`-XX:ParallelGCThreads=n`控制线程的数量。并行垃圾回收也被称为批量（throughput）回收，因为它使用多个CPU提升GC的性能。并行回收使用单个线程处理Major GC。

3. **并发Parallel Old GC (-XX:+UseParallelOldGC**：除了Minor和Marjor GC都是用多线程来处理的之外，跟Parallel GC一样。

4. **Concurrent Mark Sweep (CMS) Collector (-XX:+UseConcMarkSweepGC)**: CMS回收也被称为并发短暂停回收。他是用来处理老年代垃圾回收的。CMS回收尝试最短暂停的垃圾回收，通过使用应用的线程并行的处理大多数垃圾回收工作。CMS回收在处理年轻代回收时使用和parallel collector相同的算法。这种垃圾回收方法适用于对时延要求较高的服务。我们能用JVM配置`-XX:ParallelCMSThreads=n` 指定CMS回收的线程数量。

5. **G1 Garbage Collector (-XX:+UseG1GC)**: 从Java7开始有了G1回收，他长期的目标用来替换CMS回收。它同CMS相比，在以下方面表现的更出色： 

   - G1是一个有整理内存过程的垃圾收集器，不会产生很多内存碎片。 
   - G1的Stop The World(STW)更可控，G1在停顿时间上添加了预测机制，用户可以指定期望停顿时间。

   传统的GC收集器将连续的内存空间划分为新生代、老年代和永久代（JDK 8去除了永久代，引入了元空间Metaspace），这种划分的特点是各代的存储地址（逻辑地址，下同）是连续的。如下图所示：![传统GC内存布局](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/8a9db36e.png)

   而G1的各代存储地址是不连续的，每一代都使用了n个不连续的大小相同的Region，每个Region占有一块连续的虚拟内存地址。如下图所示：![g1 GC内存布局](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/8ca16868.png)

详情参见[Java Hotspot G1 GC的一些关键技术 - 美团技术团队 (meituan.com)](https://tech.meituan.com/2016/09/23/g1.html)

## 内存管理在Java-Java垃圾回收监控

我们可以使用Java命令行工具监控垃圾回行为，例如

```shell
java -Xmx120m -Xms30m -Xmn10m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseSerialGC -jar Java2Demo.jar
```

#### jstat

我们可以用jstat命令行工具监控JVM内存和垃圾回收行为。

首先使用 `ps -eaf | grep java` 获取应用的进程ID，然后就可以用jstat命令如下所示：

```
p2yh@mac:~$ jstat -gc 9582 1000
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT
1024.0 1024.0  0.0    0.0    8192.0   7933.3   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8026.5   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8030.0   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8122.2   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  0.0    0.0    8192.0   8171.2   42108.0    23401.3   20480.0 19990.9    157    0.274  40      1.381    1.654
1024.0 1024.0  48.7   0.0    8192.0   106.7    42108.0    23401.3   20480.0 19990.9    158    0.275  40      1.381    1.656
1024.0 1024.0  48.7   0.0    8192.0   145.8    42108.0    23401.3   20480.0 19990.9    158    0.275  40      1.381    1.656
```

最后一个 参数是每次输出的时间间隔。上文将每秒打印一次内存和垃圾回收数据。

我们逐列说明：

- **SOC and S1C**：当前S0和S1区域大小单位KB。
- **S0U and S1U**: 当前S0和S1的使用空间单位KB。可以注意到其中一个survivor始终为空。
- **EC and EU**: 当前Eden空间的总空间和使用空间单位是KB。可以注意到EU大小是增长的，当超过EC大小，Minor GC触发EU空间减小。
- **OC and OU**: 老年代的总空间和使用空间单位KB。
- **PC and PU**: 永久代的总空间和使用空间单位KB。
- **YGC and YGCT**: YGC展示了年轻代发生的GC事件次数。YGT展示了年轻代发生的GC操作的累积时间。
- **FGC and FGCT**: FGC展示Full GC的时间次数。 FGCT展示Full GC操作花费的时间。
- **GCT**: GC操作的总是耗时。可以看到它是YGCT和FGCT的求和。

#### Java VisualVM with Visual GC

可以使用可视化工具**jvisualvm**。

### Java Garbage Collection Tuning

垃圾回收调优应该是最后的选项，当你的看到超时原因是因为GC长时间运行。

假如你看到`java.lang.OutOfMemoryError: PermGen space` 的错误，那么你应该去试着监控和提高永久代内存空间通过**-XX:PermGen and -XX:MaxPermGen**。你可以也可以试着使用`-XX:+CMSClassUnloadingEnabled` 和检查使用CMS垃圾回收的运行效果。

假如你看到很多Full GC操作，那么你应该提升老年代的内存空间。

总体来看垃圾回收调优会操作很多的影响，会花费很多时间，没有固定的和快速的规则。你需要尝试不同的选项然后比较他们来找出最适合你应用的那个。