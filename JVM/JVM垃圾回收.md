### 标记算法

#### 引用计数

存在循环引用导致无法释放的问题

#### 可达性分析算法

通过GC Roots的对象作为起点，从这些节点开始向下搜索，搜索走过的路径被称为（Reference Chain），当一个对象到GC Roots没有任何引用链相连时，即从GC Roots节点到该节点不可达，则证明该object是不可用的。

![img](https://www.freecodecamp.org/news/content/images/2021/01/image-76.png)



在 Java 语言中，可作为 GC Root 的对象包括以下4种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象



##### 虚拟机栈（栈帧中的本地变量表）中引用的对象
此时的 s，即为 GC Root，当s置空时，localParameter 对象也断掉了与 GC Root 的引用链，将被回收。

```java
public class StackLocalParameter {
    public StackLocalParameter(String name){}
}

public static void testGC(){
    StackLocalParameter s = new StackLocalParameter("localParameter");
    s = null;
}
```

##### 方法区中类静态属性引用的对象
s 为 GC Root，s 置为 null，经过 GC 后，s 所指向的 properties 对象由于无法与 GC Root 建立关系被回收。

而 m 作为类的静态属性，也属于 GC Root，parameter 对象依然与 GC root 建立着连接，所以此时 parameter 对象并不会被回收。

```
public class MethodAreaStaicProperties {
    public static MethodAreaStaicProperties m;
    public MethodAreaStaicProperties(String name){}
}

public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("properties");
    s.m = new MethodAreaStaicProperties("parameter");
    s = null;
}
```

##### 方法区中常量引用的对象
m 既是方法区中的常量引用，也为 GC Root，s 置为 null 后，final 对象也不会因没有与 GC Root 建立联系而被回收。

```java
public class MethodAreaStaicProperties {
    public static final MethodAreaStaicProperties m = MethodAreaStaicProperties("final");
    public MethodAreaStaicProperties(String name){}
}

public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("staticProperties");
    s = null;
}
```

##### 本地方法栈中引用的对象
任何 Native 接口都会使用某种本地方法栈，实现的本地方法接口是使用 C 连接模型的话，那么它的本地方法栈就是 C 栈。当线程调用 Java 方法时，虚拟机会创建一个新的栈帧并压入 Java 栈。然而当它调用的是本地方法时，虚拟机会保持 Java 栈不变，不再在线程的 Java 栈中压入新的帧，虚拟机只是简单地动态连接并直接调用指定的本地方法。

### 怎么回收垃圾

#### 标记清除算法（Mark-Sweep）

- 过程：标记->整理->清除

- 缺点：产生内存碎片

#### 复制算法（Copying）

- 过程：

1. 首先将可用内存按容量划分为大小相等的两块
2. 每次只使用其中的一块
3. 当这一块的内存用完了，就将还存活着的对象复制到另外一块上面
4. 然后再把已使用过的内存空间一次清理掉

- 优点

  保证了内存的连续可用，内存分配时也就不用考虑内存碎片等复杂情况，逻辑清晰，运行高效

- 缺点

  合着我这140平的大三房，只能当70平米的小两房来使？代价实在太高。

#### 标记整理算法（Mark-Compact）

- 过程：标记->整理->清除

  ![img](https://www.freecodecamp.org/news/content/images/2021/01/image-82.png)

  ![img](https://www.freecodecamp.org/news/content/images/2021/01/image-83.png)

  ![img](https://www.freecodecamp.org/news/content/images/2021/01/image-83.png)

- 优点：解决了内存碎片的问题

- 缺点：需要频繁变动内存，效率较低

#### 分代收集算法（Generational Collection）

融合上述3种基础的算法思想，针对不同情况采用不同算法。

对象存活周期的不同将内存划分为几块。一般把 Java 堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。

- 在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。
- 老年代中对象存活率高、没有额外空间对它进行分配担保，就必须使用**标记-清理**或者**标记-整理**算法来进行回收。

那么，另一个问题来了，那内存区域到底被分为哪几块，每一块又有什么特别适合什么算法呢？

![Java Memory Model, JVM Memory Model, Memory Management in Java, Java Memory Management](https://cdn.journaldev.com/wp-content/uploads/2014/05/Java-Memory-Model-450x186.png)



### 垃圾回收类型

#### 串行Serial GC (-XX:+UseSeralGC)

Serial GC使用简单的**mark-sweep-compact**处理年轻和老年代垃圾回收也就是Minor和Marjor GC。Serial GC是在客户端机器例如我们简单的独立应用和小CPU的机器。它对运行在低内存空间的小型应用是很好用的。

![img](https://www.freecodecamp.org/news/content/images/2021/01/image-68.png)

所有的回收事件都是在一个线程上顺序执行的。每次回收结束后进行整理工作。

会导致Stop the World事件。

#### 并发Parallel GC (-XX:+UseParallelGC)

Parallel GC和Serial GC一样，除了它为Minor GC创建N个线程，N是操作系统的CPU核心数。我们可以用JVM配置`-XX:ParallelGCThreads=n`控制线程的数量。并发垃圾回收也被称为批量（throughput）回收，因为它使用多个CPU提升GC的性能。

![img](https://www.freecodecamp.org/news/content/images/2021/01/image-66.png)

并发回收使用多个线程处理Minor GC，单个线程处理Major GC。

会导致Stop the World事件。

#### 并发Parallel Old GC (-XX:+UseParallelOldGC

除了Minor和Marjor GC都是用多线程来处理的之外，跟Parallel GC一样。

#### Concurrent Mark Sweep (CMS) Collector (-XX:+UseConcMarkSweepGC)

CMS回收也被称为并行短暂停回收。CMS回收尝试最短暂停的垃圾回收，通过使用应用的线程并行的处理大多数垃圾回收工作。这种垃圾回收方法适用于对时延要求较高的服务。我们能用JVM配置`-XX:ParallelCMSThreads=n` 指定CMS回收的线程数量。

![img](https://www.freecodecamp.org/news/content/images/2021/01/image-67.png)

- 处理Minor GC时使用多线程处理，和parallel collector相同的算法。

- 处理Major GC时也是多线程，和Parallel Old GC一样。

- 但是CMS使用的是和应用线程以外的线程和应用并行执行。最小化Stop the World事件。

#### G1 Garbage Collector (-XX:+UseG1GC)

从Java7开始有了G1回收，它设计的目的是为那些具有大Heap空间（多于4GB）的多线程应用代替CMS。它同CMS相比，在以下方面表现的更出色： 

- G1是一个有整理内存过程的垃圾收集器，不会产生很多内存碎片。 
- G1的Stop The World(STW)更可控，G1在停顿时间上添加了预测机制，用户可以指定期望停顿时间。

尽管G1也是分代回收，但是他不在分区为年轻和老年代。作为代替，每一个代是一组散列的区域，这样可以支持年轻代动态调整大小。

它将heap分割为一组散列的同等大小的区域（1MB-32MB-依赖于heap的大小），然后使用多线程扫描他们。一个区域在程序运行期间的任何时候都可能是老年区域或者年轻区域。

在Mark阶段（phase）完成后，G1知道哪个区域包含最多 的垃圾objects。如果用户关心的是最小暂停时间，G1会选择清空仅仅一小部分区域。如果用户不太担心暂停时间or has stated a fairly large pause-time goal, G1可能选择清理更多的区域。

![img](https://www.freecodecamp.org/news/content/images/2021/01/image-88.png)

除了Eden，Survivor和老年内存区域。G1 GC还要有两个类型的区域

- 超大-使用超多空间的objects（大于50%heap大小）
- 可用-无用或者为申请的空间