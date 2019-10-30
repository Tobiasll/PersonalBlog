---
title: java 内存结构
date: 2019/10/07 15:01:25
---



# 序言

虽然Java和C++都属于一种面向对象型的语言，但是在内存管理方面却有着非常大的区别，书上用了《围城》中的：墙内的人想出去，墙外的人想进来这句话来形容这堵高墙，因为Java有着独特的内存动态分配策略和垃圾收集器等技术，导致Java开发人员不会像C/Cpp那样要去关注内存，从而把内存控制权力交给了jvm，使得一旦发生内存泄漏和溢出，如果对于jvm管理内存不熟悉的话，将会导致很难排查问题。



# 运行时数据区域

jvm在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。

> “Java 虚拟机具有一个堆，堆是运行时数据区域，所有类实例和数组的内存均从此处分配。堆是在 Java 虚拟机启动时创建的。”“在 JVM 中堆之外的内存称为非堆内存 (Non-heap memory)”。可以看出 JVM 主要管理两种类型的内存：堆和非堆。简单来说堆就是 Java 代码可及的内存，是留给开发人员使用的；非堆就是 JVM 留给 自己用的，所以方法区、JVM 内部处理或优化所需的内存 (如 JIT 编译后的代码缓存)、每个类结构 (如运行时常数池、字段和方法数据) 以及方法和构造方法 的代码都在非堆内存中。
>
> ![jvm_memory](/blog_image/jvm_text1/jvm_memory.png)
>
> ![3d0db09e499f8436e762e8f1072825be_663x613](/blog_image/jvm_text1/3d0db09e499f8436e762e8f1072825be_663x613.png)
>
> 

JAVA 的 JVM 的内存可分为 3 个区：堆区（堆内和堆外）、栈 区(虚拟机栈和本地方法栈) 和方法区 (method)

## 程序计数器（pc寄存器）

一块比较小的内存空间，可以看作当前线程所执行的字节码的行号指示器。在虚拟机规范中指pc寄存器。

- 当正在执行的方法为native时，这个计数器值则为空。
- 当该方法不是native时，就保存jvm正在执行的字节码指令的**地址**。
- 每一条jvm线程都有直接的程序计数器（线程私有）。
- 任意时刻，一条jvm线程只会执行一个该线程的当前方法。
- 保证至少可以存一个returnAddress类型的数据或本地指针的值
- 当前线程所执行的字节码行号指示器。
- 此内存区域是唯一一个在 Java 虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。

## JAVA 虚拟机栈

Java虚拟机栈描述的是Java方法执行的内存模型（JMM），并且每个方法执行的同时都会创建一个栈帧，因为除了栈帧的出入栈之外，Java虚拟机栈不会再受到其他因为的影响，所以栈帧可以在堆中分配。

- 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 StackOverflowError 异常。
- 如果虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出 OutOfMemoryError 异常。
- 是线程私有的。
- Java虚拟机也不需要保证内存的连续性。
- 通过 -Xss 调整线程堆栈大小，1.5 之后为 1M，之前为 256k，减少堆栈大小，可创建更多线程。

## 本地方法栈

与Java虚拟机栈执行Java方法（字节码）服务不同，本地方法栈是jvm使用到的**Native**方法服务

- 与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。
- Nativa方法本质上是依赖于具体实现的
- 可以自由的决定使用怎样的机制来让Java程序调用本地方法
- 任何本地方法接口都会使用某种本地方法栈
- 本地方法接口拥有和jvm相同的能力，可以访问虚拟机运行时数据区、直接使用本地处理器的寄存器、直接从本地内存的堆中分配人员数量的内存...
- jvm规范允许本地方法栈实现成固定大小或者根据计算来动态拓展和收缩

> Figure 5-13 shows a graphical depiction of a thread that invokes a native method that calls back into the virtual machine to invoke another Java method. This figure shows the full picture of what a thread can expect inside the Java Virtual Machine. A thread may spend its entire lifetime executing Java methods, working with frames on its Java stack. Or, it may jump back and forth between the Java stack and native method stacks.
>
> ![8859852a-04cf-3ad1-871f-3f9126741580](/blog_image/jvm_text1/8859852a-04cf-3ad1-871f-3f9126741580.jpg)
>
> As depicted in Figure 5-13, a thread first invoked two Java methods, the second of which invoked a native method. This act caused the virtual machine to use a native method stack. In this figure, the native method stack is shown as a finite amount of contiguous memory space. Assume it is a C stack. The stack area used by each C-linkage function is shown in gray and bounded by a dashed line. The first C-linkage function, which was invoked as a native method, invoked another C-linkage function. The second C-linkage function invoked a Java method through the native method interface. This Java method invoked another Java method, which is the current method shown in the figure.

## 堆(堆内)

java heap是可供各个线程**共享**的运行时内存区域，也是供所有类实例和数组对象分配内存的区域，存储了GC所管理的各种对象，内存泄漏最容易发生的区域

- 如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

- 基本上都采用了分代收集算法，分为新生代、老年代、其中新生代还分为Eden(80%)、From Survivor(10%)、To survivor(10%)

  > **新生代**： 程序新创建的对象都是从新生代分配内存，新生代由 Eden Space 和两块相同大小的 Survivor Space (通常又称 S0 和 S1 或 From 和 To) 构成，可通过 - Xmn 参数来指定新生代的大小，也可以通过 - XX:SurvivorRation 来调整 Eden Space 及 Survivor Space 的大小。 **老年代**： 用于存放经过多次新生代 GC 任然存活的对象，例如缓存对象，新建的对象也有可能直接进入老年代，主要有两种情况：①. 大对象，可通过启动参数设置 - XX:PretenureSizeThreshold=1024 (单位为字节，默认为 0) 来代表超过多大时就不在新生代分配，而是直接在老年代分配。②. 大的数组对象，切数组中无引用外部对象。 老年代所占的内存大小为 - Xmx 对应的值减去 - Xmn 对应的值。
  > ![img](/blog_image/jvm_text1/5c113a485acbeb7888cf08701bcd595d_526x298.png)
  >
  > | 区域                 | 描述                                   |
  > | -------------------- | -------------------------------------- |
  > | Young Generation     | 即图中的 Eden + From Space + To Space  |
  > | Eden                 | 存放新生的对象                         |
  > | Survivor Space       | 有两个，存放每次垃圾回收后存活的对象   |
  > | Old Generation       | Tenured Generation 即图中的 Old Space  |
  > | Permanent Generation | 主要存放应用程序中生命周期长的存活对象 |

- 堆容量可以是固定的也可以是动态拓展和收缩的，内存不需要保证连续

- 你可以用 JConsole 或者 Runtime.maxMemory (), Runtime.totalMemory (), Runtime.freeMemory () 来查看 Java 中堆内存的大小。

- 你可以使用命令 “jmap” 来获得 heap dump，用 “jhat” 来分析 heap dump。

- Java 堆内存是操作系统分配给 JVM 的内存的一部分。

- 请使用 Profiler 和 Heap dump 分析工具来查看 Java 堆空间，可以查看给每个对象分配了多少内存。

## 直接内存(堆外)

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError异常出现，

- 本机直接内存的分配不会受到Java堆大小的限制，但是，既然是内存，肯定还是会受到本机总内存大小以及处理器寻址空间的限制
- 配置虚拟机参数时不要忽略直接内存，因为会使得各个内存区域总和大于物理内存的限制，从而导致拓展时出现OOM异常
- 基于通道（Channel）和缓冲区（Buffer）IO方式的NIO，可以使用Native函数库直接分配堆外内存，避免在Java堆和native堆来回复制数据来提升总体的性能
- 不会影响到堆内内存大小

## 方法区

方法区是可供各个线程共享的内存区域，存储了每一个类的**结构**信息，jvm规范中容量可以是固定的也可以是动态收缩的，并且内存不用保证有连续性

- 根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。
- JDK 7之前，使用永久代实现方法区（容易遇到内存溢出问题），JDK 7 使用Native Memory本地内存来代替永久代，JDK8使用**Metaspace**元数据区来实现方法区，利用元数据分配只受本地内存大小的限制（本地内存剩多少，元数据就有多大）来解决永久代的OOM问题

## 运行时常量池

运行时常量池是class文件中每一个类或接口的常量池的运行是表示形式，存储数据的范围都比通常意义上的符号表要更广泛，都在jvm的方法区中进行分配，在加载类和接口道jvm后创建对应的运行时常量池，同时具备方法区的动态拓展和收缩

- 受到方法区内存的限制，当常量池无法再申请到内存时会抛出OutOfMemoryError异常。
- Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。

## 栈帧



# 小结

Java的内存结构的大致就分以上几个点，其中Java堆，是比较重要的一块，也是常见的内存溢出重灾区，GC主要区域，篇幅会比较多



# 参考资料

《深入理解Java虚拟机》 周志明

《Java 8 虚拟机规范》