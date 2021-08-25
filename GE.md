#### Jar中MANIFEST文件的作用

- JAR 包基本信息描述：Mainfest-Version，生成者，签名版本，类的搜索路径
- Main-Class 指定程序的入口，这样可以直接用java -jar xxx.jar来运行程序
- Class-Path 指定jar包的依赖关系，class loader会依据这个路径来搜索class

#### final，finally，finalize

- Final
  - 修饰类：
  - 属性：一经定义就不能被修改，final 修饰的引用类型，只是保证对象的引用不会改变。对象内部的数据可以改变。
  - 方法：方法不可以被重写 
- Finally
  - finally 保证程序一定被执行
- Finalize 
  - finalize 是祖宗类 `Object`类的一个方法，它的设计目的是保证对象在垃圾收集前完成特定资源的回收。

#### native关键字

一个Native Method就是一个java调用非java代码的接口。比如在C＋＋中，你可以用extern "C"告知C＋＋编译器去调用一个C的函数。

1. 在Java中声明native()方法，然后编译
2. 用javah产生一个.h文件；
3. 写一个.cpp文件实现native导出方法，其中需要包含第二步产生的.h文件（注意其中又包含了JDK带的jni.h文件）；
4. 将第三步的.cpp文件编译成动态链接库文件；
5. 在Java中用System.loadLibrary()方法加载第四步产生的动态链接库文件，这个native()方法就可以在Java中被访问了。

#### volatile

**可见性**是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

volatile可以禁止**指令重排**，这就保证了代码的程序会严格按照代码的先后顺序执行。



####  transient 

被 transient 修饰的变量不能被序列化。

1. transient修饰的变量不能被序列化；
2. transient只作用于实现 Serializable 接口；
3. transient只能用来修饰普通成员变量字段；
4. 不管有没有 transient 修饰，静态变量都不能被序列化；



#### abstract class和interface的区别

|          | Abstract class                                               | Interface                                                    |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 实例化   | 不能                                                         | 不能                                                         |
| 类       | 一种继承关系，一个类只能使用一次继承关系。可以通过继承多个接口实现多重继承 | 一个类可以实现多个interface                                  |
| 数据成员 | 可有自己的                                                   | 静态的不能被修改即必须是static final，一般不在此定义         |
| 方法     | 可以私有的，非abstract方法，必须实现                         | 不可有私有的，默认是public，abstract 类型                    |
| 变量     | 可有私有的，默认是friendly 型，其值可以在子类中重新定义，也可以重新赋值 | 不可有私有的，默认是public static final 型，且必须给其初值，实现类中不能重新定义，不能改变其值。 |
| 设计理念 | 表示的是“is-a”关系                                           | 表示的是“like-a”关系                                         |
| 实现     | 需要继承，要用extends                                        | 要用implements                                               |

 #### HashMap和TreeMap的区别

实现：HashMap 链表+红黑树；TreeMap是红黑树

有序性：HashMap无序，TreeMap有序

Null值：HashMap允许null，*TreeMap* doesn't allow a *null* *key* but may contain many *null* values.



#### synchronized 和 Lock 异同点

##### 相同点

- synchronized 和 Lock 都是用来保护资源线程安全的。
- 都可以保证可见性。
- synchronized 和 ReentrantLock 都拥有可重入的特点。

##### 不同点

- 用法区别
  - synchronized 关键字可以加在**方法**上，不需要指定锁对象（此时的锁对象为 this），也可以新建一个同步**代码块**并且自定义 monitor 锁对象；
  - 而 Lock 接口必须显示用 Lock 锁对象开始加锁 lock() 和解锁 unlock()，并且一般会在 finally 块中确保用 unlock() 来解锁，以防发生死锁。
  - 与 Lock 显式的加锁和解锁不同的是 **synchronized 的加解锁是隐式**的，尤其是抛异常的时候也能保证释放锁，但是 Java 代码中并没有相关的体现。
- synchronized 锁不够灵活
  - synchronized 锁已经被某个线程获得了，此时其他线程如果还想获得，那它只能被**阻塞**
  - Lock 类在等锁的过程中，如果使用的是 **lockInterruptibly** 方法，那么如果觉得等待的时间太长了不想再继续等待，可以中断退出，也可以用 **tryLock()** 等方法尝试获取锁，如果获取不到锁也可以做别的事，更加灵活。
  - synchronized 锁只能同时被一个线程拥有，但是 Lock 锁没有这个限制
  - 是否可以设置公平/非公平：公平锁是指多个线程在等待同一个锁时，根据先来后到的原则依次获得锁。ReentrantLock 等 Lock 实现类可以根据自己的需要来设置公平或非公平，synchronized 则不能设置。

##### 如何选择

1. 如果能不用最好既不使用 Lock 也不使用 synchronized。因为在许多情况下你可以使用 java.util.concurrent 包中的机制，它会为你处理所有的加锁和解锁操作，也就是推荐优先使用工具类来加解锁。
2. 如果 synchronized 关键字适合你的程序， 那么请尽量使用它，这样可以减少编写代码的数量，减少出错的概率。因为一旦忘记在 finally 里 unlock，代码可能会出很大的问题，而使用 synchronized 更安全。
3. 如果特别需要 Lock 的特殊功能，比如尝试获取锁、可中断、超时功能等，才使用 Lock



#### Stack和Deque区别

- 双端队列（Double Ended Queue），简称Deque

把Deque当栈用的时候：

| 操作     | 函数                                                    |
| -------- | ------------------------------------------------------- |
| 入栈     | push(E e)                                               |
| 出栈     | poll() / pop() 后者在栈空的时候会抛出异常，前者返回null |
| 查看栈顶 | peek() 为空时返回null                                   |



把Deque当队列用的时候：

| 操作     | 函数                  |
| -------- | --------------------- |
| 入队     | offer(E e)            |
| 出队     | poll() 为空时返回null |
| 查看队首 | peek() 为空时返回null |



#### JAVA中线程同步的方法

- synchronized
- wait：使一个线程处于等待状态，并且释放所持有的对象的lock。
- notify：唤醒一个处于等待状态的线程，注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且不是按优先级。
- volatile
- ReenreantLock
- ThreadLocal
- LinkedBlockingQueue：阻塞队列



#### JAVA内存泄露的？

- 长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄露

- 集合中的内存泄漏，比如 HashMap、ArrayList 等，这些对象经常会发生内存泄露。比如当它们被声明为静态对象时，它们的生命周期会跟应用程序的生命周期一样长，很容易造成内存不足。

   ```
   Vector v = new Vector(10);
   
   for (int i = 1; i < 100; i++) {
   
       Object o = new Object();
   
       v.add(o);
   
       o = null;
   
   }
   ```

  

1. 静态集合类引起内存泄漏：

2. 当集合里面的对象属性被修改后，再调用remove()方法时不起作用。

3. 各种连接

   比如数据库连接（dataSourse.getConnection()），网络连接(socket)和io连接，除非其显式的调用了其close（）方法将其连接关闭，否则是不会自动被GC 回收的。

4. 单例模式

   不正确使用单例模式是引起内存泄漏的一个常见问题，单例对象在初始化后将在JVM的整个生命周期中存在（以静态变量的方式），如果单例对象持有外部的引用，那么这个对象将不能被JVM正常回收，导致内存泄漏



##### 如果进行Java remote debug

1. 首先被debug程序的虚拟机在启动时要**开启debug模式**，启动debug监听程序。

   DEBUG选项参数的意思

   ```javascript
   -XDebug 启用调试；
   -Xrunjdwp 加载JDWP的JPDA参考执行实例；
   transport 用于在调试程序和VM使用的进程之间通讯；
   dt_socket 套接字传输；
   server=y/n VM是否需要作为调试服务器执行；
   address=7899 调试服务器监听的端口号；
   suspend=y/n 是否在调试客户端建立连接之后启动 VM 。
   ```

2. 用一个debug客户端去debug远程的程序

