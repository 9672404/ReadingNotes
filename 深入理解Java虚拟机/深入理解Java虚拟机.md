# 《深入理解Java虚拟机》

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B/%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE.png)

## 一、走进Java

+ java的发展史和各个版本发布时间及特性。

* HotSpot VM 热点代码探测能力可以通过计数器找到最具有编译价值的代码，然后通知JIT编译器已方法为单位进行编译。如果一个方法被频繁调用或方法中有效循环次数很多，将会分别触发标准编译和OSR（栈上替换）编译动作。通过编译器和解释器恰当地协调工作，可以在最优化的程序响应时间与最佳的执行效率种取得平衡，而且无需等待本地代码输出才能执行程序，即时编译的时间压力也相对减小，这样有利于引入个更多的代码优化技术，输出质量更高的本地代码。

## 二、自动内存管理机制

### 第2章  Java内存区域与内存溢出异常

![虚拟机运行时数据区](https://readingnotes.oss-cn-beijing.aliyuncs.com/%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.png)

#### 2.1 运行时数据区

1. 程序计数器（私有线程）

   * “程序计数器” 是一块较小的内存空间，它可以看那做是当前线程所执行的字节码的行号指示器，在虚拟机的概念模型里，字节码解释器工作就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
   * java 虚拟机中的多线程是通过线程轮流切换分配处理器执行时间的方式实现的。
   * 如果线程正在执行的是一个方法，这个计数器记录的是正在执行的虚拟机字节码的指令地址。
2. Java虚拟栈（私有线程）

   - 描述的是Java方法执行的内存模型：每个方法在执行的同时，都会创建一个栈帧（Stack Frame）用于储存局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的操作。
   - 局部变量表存放了编译期间可知的各种基础数据类型（boolean、byte、char、short、int、float、long、double）、对象类型（不等于对象本身，有可能是一个指向对象其实地址的引用指针）和returnAddress类型（指向了一条字节码指令的地址）
3. 本地方法栈

   * 执行Native方法，Native方法实现的语言没有规定。Hotspot直接把虚拟机栈和本地方法栈合二为一，会抛出 StackOverflowError 和 OutOfMemoryError 异常
4. Java堆（线程共享）

   * 次区域的作用就是存放对象实例（数组），随着JIT编译技术发展和逃逸分析的成熟，所有对象都分配在堆上也就没有那么绝对了。
   * 垃圾收集器一般采用分代收集算法，所有堆又可以分为 新生代、老年代。
   * 分配的内存空间物理上可以不连续，逻辑上连续就可以。可以扩展堆内存大小，如果堆中没有内存完成分配并且已已经无法扩展就会抛出，OutOfMemoryError 异常。
5. 方法区（线程共享）
   * 用于储存被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，java虚拟机规范把方法区描述成堆的一个逻辑部分，但是它却有个别名就是 “非堆”，目的是与Java堆分开。
   * 方法区又可以成为“永久代”，但是两者不等价，是因为HotSpot 把GC分代收集扩展至方法区，但是这部分的内存所以随着类加载分配的，生命周期为类的生命周期，一般不会被收集，所以被称为永久代。由于内存得不到有效的回收，容易出现OutOfMemoryError 异常。
6. 运行时常量池
   * 是方法区的一部分，用于存放编译期生成的字面量和符号引用，这部分内容将在类加载后进入方法区的运行常量池用存放。
   * 除了保存Class文件中描述的符号引用外，还会把翻译出来的直接引用也储存的运行时常量池中。

#### 2.2 HotSpot虚拟机对象探秘

1. 对象的创建
   * 虚拟机遇到一条new指令时，首先将检查这个指令的参数能否在常量池中定位到一个类的符号引用，并检查这个符号引用代表的类是否已被加载、解析、初始化过，如果没有则执行相应的类加载过程。
   * 在类加载检查通过后，虚拟机将为新生代对象分配内存（对象所需内存大小在类加载完便完全确认），划分方式可以分为“指针碰撞”就是通过指针移动划分出一整块新空间；"空闲列表"分配不连续空间，从空闲列表上查询分配哪些空间。
   * 对象的划分（指针的移动）涉及到线程安全问题。一种是对分配内存空间的操作进行同步处理—实际上虚拟机采用的CAS配上失败重试的方式保证更新操作的原子性；另一种方式是把内存分配的动作按照线程划分在不用的空间内进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（TLAB）。需要分配空间现在本地线程缓冲区中分配，只有再重新分配TLAB的时候才涉及到同步操作。
   * 虚拟机需要将分配的空间都初始化为0，如果是TLAB 这个操作提前到TLAB分配时进行，这一步保证了对象不赋初始值可以访问对象里的属性空值或0值，而不是空指针。
   * 设置对象的对象头：对象的哈希码、对象的GC分代年龄等信息。根据虚拟机运行状态不同，如是否启用偏向锁，对对象头有不同的设置方式。
   * 执行完new指令后会接着执行<init> 方法，对对象进行初始化。

2. 对象的内存分布

   * 在HotSpot虚拟机中，对象在内存中的储存可分为三块区域，对象头、实例数据、对齐填充。

   * 对象头：

     > 第一部分：储存自身的运行时数据，如 哈希码（hashCode）、GC分代年龄、锁状态标志、偏向线程ID、偏向时间戳。在32位和64位的虚拟机中分别占32bit和64bit。考虑到虚拟机的空间效率，对象头被设计成非固定的数据结构以便在较小的空间内储存尽量多的信息。
     >
     > 第二部分：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。如果是Java数组，在对象头中还必须有一块记录数组长度的数据。虚拟机可以通过对象的元数据来确定大小，但是数组无法确定。

   * 实例数据:

     > 储存的是对象的有效数据，为类中定义的各种类型的字段，无论是父类继承的还是子类定义的，这部分存储的顺序会受到虚拟机分配策略的参数和字段在Java源码中定义的顺序的影响。
     >
     > 默认分配策略为 longs/double 、ints、shorts/chars、bytes/booleans、oops、从分配策略上看相同宽度的字段被分配在一起。

   * 对齐填充：

     > 并不是必然存在，没有特殊意义，只是起到占位符的作用。

3. 对象的访问定位

   * 句柄访问：对象的引用存储的是对象的句柄地址，再根据句柄地址再根据句柄存放的地址访问真正实例的地址。
   * 直接指针访问：速度快、节省了一次指针定位时间。HotSpot使用这种方式。 

### 第3章 垃圾收集器与内存分配策略

#### 3.2 对象已死吗？

##### 3.2.1 引用计数法：

	给对象添加一个计数器，有引用时 +1  ，失去引用时 -1；特点：实现简单，判断效率快，但是很难解决对象之间相互循环引用的问题。

##### 3.2.2 可达性分析算法：

	主流的JVM实现，从GC Roots 作为起点，从这些节点开始往下搜索，搜索的路径叫做引用链，如果一个对象到GC Root没有任何引用链相连，则说明该对象没有被引用。如图：

Object5,6,7 虽然相互关联，但是没有到GC Root是不可达的，所以可以回收。

![可达性算法分析对象是否可回收 ](https://readingnotes.oss-cn-beijing.aliyuncs.com/%E3%80%8A%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E3%80%8B/%E5%8F%AF%E8%BE%BE%E6%80%A7%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%E5%AF%B9%E8%B1%A1%E6%98%AF%E5%90%A6%E5%8F%AF%E5%9B%9E%E6%94%B6.png)

* 可作为GC Roots的对象有

> 虚拟机栈中（栈帧的本地变量表）引用的对象。
>
> 方法区中静态属性引用的对象。
>
> 方法区中常量引用的属性。
>
> 本地方法栈中JNI(即一般说的(Native方法)引用的对象。

##### 3.2.3 再谈引用

JDK1.2定义：如果reference 类型的数据中储存的是另一块内存的起始位置，则说明这块内存代表着一个引用。

还希望能有一些对象，如果内存充足则存在内存中，如果进行GC后内存还紧张，对象就销毁，各种系统缓存都是此场景。JDK1.2后对引用的概念进行扩充。

* 强引用：类似 Object obj = new Object();的引用，只要引用存在，被引用的对象就不会被回收。
* 软引用：用来描述一些有用但非必须的对象，在系统要引起内存溢出异常的时候，先对软引用的对象进行进行回收，如果还没有足够内存，则抛出内存溢出异常。SoftReference来实现。
* 弱引用：用来描述非必须的对象，在垃圾收集器工作时，无论内存是否充足，只关联软引用的对象都会被回收。WeakReference来实现。
* 虚引用：虚引用完全不会对对象的生命周期产生影响，唯一的用处就是对象被垃圾回收的时候可以得到一个通知。PhantomReference实现。

##### 3.2.4 生存还是死亡？

* 真正的对象回收一般涉及到对象的两次标记，如果经过可达性分析后，对象没有和GC Roots建立连接，则被标记第一次，并进行筛选 如果对象没有覆盖了finalize()方法或该对象的finalize()方法已经被虚拟机调用过一次，判定没有必要执行，则直接进行回收；否则判定有必要执行，把有必要执行finalize（）方法的对象放到一个 F-queue 的队列中，虚拟机会自启一个低优先级的线程去执行队列中的对象的 finalize()方法，这里只是触发方法，不保证方法能运行完（如果方法执行慢或逻辑异常，队列中的其他对象会一直等待），然后对队列中的对象进行二次标记，如果这时 对象能和GC Roots进行关联，则被移除“即将回收”的集合，但是此对象方法虚拟机只会触发一次，如果对象第二次被标记可回收，则不会调用finalize()方法。
* finalize()方法只是为了java设计时为了让C/C++ 程序员更好的理解添加的方法（类C++中的析构函数），可以做外部资源关闭等操作，但是不推荐使用。



##### 3.2.5 回收方法区

方法区也被叫做永久代，主要涉及废弃常量和无用类的回收

* 回收常量只需要判断系统中有没有一个String对象引用到常量池中的某常量，如果没有则发生回收。

* 判断无用类的条件

  > * 该类所有的实例都被回收。
  > * 加载该类的ClassLoader被回收。
  > * 该类对应的 java.lang.Object 对象没有在任何地方被应用，无法在任何地方通过反射访问到类中的方法。

  即使满足了条件，也不一定会发生回收，可以通过 -Xnoclassgc  参数来控制不回收。

  在频繁使用自定义ClassLoader的场景，需要这种类卸载机制，来保证永久代的内存不会发生溢出。

#### 3.3 垃圾收集算法

##### 3.3.1 标记-清理算法

	操作：首先要标记处所有需要回收的对象，并统一回收。

​	不足：标记和清理的效率不高，容易产生不连续的内存，在后面需要分配大的内存的时候，不得不进行一次垃圾回收。

#####  3.3.1 复制算法

	操作：将可用内存分成两块，每次只使用其中一块，当一块使用完了，把这块还存活的对象复制到另一块，直接清除这块内存的空间。

	不足：清理操作简单，效率高，但是空间成本大，将内存变为可使用的一半。

	经过统计，新生代对象98%都是第一次GC就回收了，所以不需要空间平均分 ，而是将空间分成 一块大的Eden(80%) 和两块较小的 Survivor(10%)，称为From和To区 ,在创建对象的时候只使用 Eden区和一个Survivor区，当发生垃圾回收时，将存活的对象复制到 Survivor的To区，如果对象被标记次数多了就会移到老年区，然后清理剩余两块空间，并复制To区到From区，如果Survivor区空间不够了，会通过新生代内存分配担保直接进入到老年代。

###### 3.3.3 标记-整理算法







