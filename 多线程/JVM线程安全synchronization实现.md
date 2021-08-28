### Object and class locks

所有线程共享的数据存储在JVM中的两个内存区域

- 堆，保存所有的objects
- 方法区，保存所有的类变量

为了保证多线程访问共享数据的安全，JVM使用锁来确保同一时刻只有一个线程能够执行。

### Monitors

JVM使用的锁是结合monitors的。一个monitor可以看成一个guardian，监控一串的代码，确保在一时刻只能有一个线程执行代码。

每个monitor关联一个object的引用。当一个线程到达代码块的第一个monitor指令，线程必须获取（obtain）引用object的锁。线程在获取到锁之前不允许执行代码。一旦他获取到锁，线程就进入被保护的代码块。

当线程离开代码块，不论是如何离开，它释放关联（associated）object的锁。

### 理解Java对象头与Monitor

在JVM中，对象在内存中的布局分为三块区域：对象头、实例数据和对齐填充。如下：

- 对象头：Java头对象，它实现synchronized的锁对象的基础
- 实例变量：存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。
- 填充数据：由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐。

一般而言，synchronized使用的锁对象是存储在Java对象头里的，jvm中采用2个字来存储对象头(如果对象是数组则会分配3个字，多出来的1个字记录的是数组长度)，其主要结构是由Mark Word 和 Class Metadata Address 组成，其结构说明如下表：

| 虚拟机位数 | 头对象结构             | 说明                                                         |
| ---------- | ---------------------- | ------------------------------------------------------------ |
| 32/64bit   | Mark Word              | 存储对象的hashCode、锁信息或分代年龄或GC标志等信息           |
| 32/64bit   | Class Metadata Address | 类型指针指向对象的类元数据，JVM通过这个指针确定该对象是哪个类的实例。 |

其中Mark Word在默认情况下存储着对象的HashCode、分代年龄、锁标记位等以下是32位JVM的Mark Word默认存储结构

| 锁状态   | 25bit                        | 4bit         | 1bit是否是偏向锁 | 2bit 锁标志位 |
| -------- | ---------------------------- | ------------ | ---------------- | ------------- |
| 偏向锁   | 线程ID\Epoch                 | 对象分代年龄 | 1                | 01            |
| 轻量级锁 | 指向栈中锁记录的指针         | 对象分代年龄 | 0                | 00            |
| 重量级锁 | 指向互斥量（重量级锁）的指针 | 对象分代年龄 | 0                | 10            |
| GC标记   | 空                           | 空           | 空               | 11            |



### Multiple locks

单个线程是允许对同一个对象加锁的。引用计数控制锁。

### Synchronized blocks

Java将多线程同时访问共享变量的情况叫做*synchronization*。Java提供两种内建的同步访问方法：

- 同步代码块（Synchronized statements）
- 同步方法 （Synchronized methiods）

#### 同步代码块-Synchronized statements

```java
class KitchenSync {
    private int[] intArray = new int[10];
    void reverseOrder() {
        synchronized (this) {
            int halfWay = intArray.length / 2;
            for (int i = 0; i < halfWay; ++i) {
                int upperIndex = intArray.length - 1 - i;
                int save = intArray[upperIndex];
                intArray[upperIndex] = intArray[i];
                intArray[i] = save;
            }
        }
    }
}
```

上面的代码，被synchronized保护的块将不执行，直到锁被当前object（this）获取(acquired)到。

Two opcodes(操作码/指令), `monitorenter` and `monitorexit`, are used for synchronization blocks within methods, as shown in the table below.

**Table 1. Monitors**

| Opcode         | Operand(s) | Description                                               |
| :------------- | :--------- | :-------------------------------------------------------- |
| `monitorenter` | none       | pop objectref, acquire the lock associated with objectref |
| `monitorexit`  | none       | pop objectref, release the lock associated with objectref |

#### 同步方法-Synchronized methods

```java
class HeatSync {
    private int[] intArray = new int[10];
    synchronized void reverseOrder() {
        int halfWay = intArray.length / 2;
        for (int i = 0; i < halfWay; ++i) {
            int upperIndex = intArray.length - 1 - i;
            int save = intArray[upperIndex];
            intArray[upperIndex] = intArray[i];
            intArray[i] = save;
        }
    }
}
```

JVM不使用任何特殊的指令去调用或者返回从Synchronized methods。当JVM决定/分解/解析（resolves）一个方法象征引用时，它查明（determines）是否方法是Synchronized。如果是，JVM在执行方法前申请锁。

- 对于一个instance方法，JVM请求获取方法开始执行之上的关联对象的锁。

- 对于一个class方法，它请求获取方法所属class关联的锁。

同步方法完成之后，无论它是正常返回或者抛异常锁都会被释放。



###  Lock 相比优缺点

- synchronized自动释放锁，Lock必须显式的释放锁。
- Lock灵活，
  - 可被多个线程持有，synchronized只能被一个线程拥有。
  - Lock可以设置为公平锁
  - 解锁顺序可变更



