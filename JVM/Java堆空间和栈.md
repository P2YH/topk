### Java堆空间

Java Heap空间用于运行时给Objects和JRE classes申请内存。无论何时我们创建object，他都会在Heap上被创建。

垃圾回收在堆上执行以便于释放哪些不再被引用的objects所占用的内存。任何在Heap空间上创建的object都能被全局访问，可以被应用在任何地方引用。

### Java栈内存

Java Stack被线程执行使用的。这就包括方法指定变量short-lived，方法引用heap上的objects。

Stack内存引用是LIFO（Last-In-First-Out）后进先出。无论何时方法被调用，新的block在stack内存上被创建，供方法持有本地变量和引用其他objects。

当方法结束时，block变为unused，然后可以供下一个方法使用。

Stack内存大小相对于Heap内存时非常少的。



### 堆和栈内存在JAVA程序

让我们通过如下简单的代码理解堆和栈内存的使用。

```java
package com.journaldev.test;

public class Memory {

	public static void main(String[] args) { // Line 1
		int i=1; // Line 2
		Object obj = new Object(); // Line 3
		Memory mem = new Memory(); // Line 4
		mem.foo(obj); // Line 5
	} // Line 9

	private void foo(Object param) { // Line 6
		String str = param.toString(); //// Line 7
		System.out.println(str);
	} // Line 8

}
```

下图展示了堆和栈如果存储私有，Objects和引用变量的。

![java memory management, java heap space, heap vs stack, java heap, stack vs heap](https://www.journaldev.com/wp-content/uploads/2014/08/Java-Heap-Stack-Memory-450x254.png.webp)

- 在程序启动的时候，它加载所有运行时classes到Heap。当main()方法在第一行被发现时，Java运行时创建stack内存用来供main()方法线程使用。
- 我们在line2创建一个私有局部变量，所以它被创建和存储在main()方法的stack内存。
- 因我我们在line3创建了一个Object，它在heap内存创建，在stack内存中引用。他和line的new Memory object一样。
- 现在当我们在line5调用foo()方法时，在栈顶创建一个block供foo()方法栈空间使用。
- 在line7创建了一个string，它是存储在String Pool中的，存在堆空间，被foo()方法stack空间引用。
- 在line8时foo()方法结束，在这时为foo()方法申请的block被释放。
- 在line9,main()方法结束，main()方法的栈空间销毁。同时程序在这行结束，hence Java运行时释放所有的内存，然后结束程序。

### 堆和栈的区别

1. 堆内存是是应用的所有部分使用的，栈仅仅供一个线程执行。
2. 无论何时一个object被创建，它总是存储在Heap空间，栈内存持有它的引用。栈内存仅仅持有局部变量和堆objects的引用。
3. 存储在堆上的Objects全局可访问，栈内存不能被其他线程访问。
4. stack内存管理是LIFO。堆内存管理要更复杂，因为堆是全局访问的。Heap内存时分为年轻代和老年代等。
5. 栈内存时short-lived，堆生命周期从应用开始到结束。
6. 我们可以使用JVM选项**-Xms** 和**-Xmx**来定义启动大小和最大的堆内存大小。我们可以使用**-Xss**来定义栈内存大小。
7. 当栈内存空间满了的时候，Java运行时抛出`java.lang.StackOverFlowError`。如果堆内存空间满了， 它抛出 `java.lang.OutOfMemoryError: Java Heap Space` 错误。
8. 栈内存大小比堆内存小得多。因为使用简单的LIFO内存分配，栈内存同堆对比是非常快速的。

