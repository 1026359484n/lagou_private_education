# JVM复盘



**jvm主要分为类装载器子系统、运行时数据区、执行引擎、本地方法接口和垃圾收集 这五大模块。 我们一般比较关注的事jvm帮我们进行的内存管理的这部分，也就是运行时数据区和垃圾收集模块**

## 1. Java 内存区域详解

如果没有特殊说明，都是针对的是 HotSpot 虚拟机。

### 1.1 概述

对于 Java 程序员来说，在虚拟机自动内存管理机制下，不再需要像 C/C++程序开发程序员这样为每一个 new 操作去写对应的 delete/free 操作，不容易出现内存泄漏和内存溢出问题。正是因为 Java 程序员把内存控制权利交给 Java 虚拟机，一旦出现内存泄漏和溢出方面的问题，如果不了解虚拟机是怎样使用内存的，那么排查错误将会是一个非常艰巨的任务。

### 1.2 运行时数据区域

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。JDK 1.8 和之前的版本略有不同，下面会介绍到。

**JDK 1.8 之前：**

[![JVM运行时数据区域](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/JVM运行时数据区域.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/pictures/java内存区域/JVM运行时数据区域.png)

**JDK 1.8 ：**

[![img](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/Java运行时数据区域JDK1.8.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/pictures/java内存区域/Java运行时数据区域JDK1.8.png)

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)

#### 1.2.1 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。**



另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**

**从上面的介绍中我们知道程序计数器主要有两个作用：**

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意：程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**

#### 1.2.2 Java 虚拟机栈

**与程序计数器一样，Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。**

**Java 内存可以粗糙的区分为堆内存（Heap）和栈内存 (Stack)，其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。** （实际上，Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。）

**局部变量表主要存放了编译期可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。**

- **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **`OutOfMemoryError`：** Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

[![img](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/《深入理解虚拟机》第三版的第2章-虚拟机栈.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/pictures/java内存区域/《深入理解虚拟机》第三版的第2章-虚拟机栈.png)

Java 虚拟机栈也是线程私有的，每个线程都有各自的 Java 虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

**扩展：那么方法/函数如何调用？**

Java 栈可以类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java 方法有两种返回方式：

1. return 语句。
2. 抛出异常。

不管哪种返回方式都会导致栈帧被弹出。

#### 1.2.3 本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误。

#### 1.2.4 堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 世界中“几乎”所有的对象都在堆中分配，但是，随着 JIT 编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。从 JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC 堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

在 JDK 7 版本及 JDK 7 版本之前，堆内存被通常分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永生代(Permanent Generation)

[![JVM堆内存结构-JDK7](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/JVM堆内存结构-JDK7.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/pictures/java内存区域/JVM堆内存结构-JDK7.png)

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。



[![JVM堆内存结构-JDK8](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/JVM堆内存结构-jdk8.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/pictures/java内存区域/JVM堆内存结构-jdk8.png)

**上图所示的 Eden 区、两个 Survivor 区都属于新生代（为了区分，这两个 Survivor 区域按照顺序被命名为 from 和 to），中间一层属于老年代。**

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

> “Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的一半时，取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
> size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> total += sizes[age];//sizes数组是每个年龄段对象大小
> if (total > desired_survivor_size) break;
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
> ```

堆这里最容易出现的就是 OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当 JVM 花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发此错误。(和配置的最大堆内存有关，且受制于物理内存大小。最大堆内存可通过`-Xmx`参数配置，若没有特别配置，将会使用默认值，详见：[Default Java 8 max heap size](https://stackoverflow.com/questions/28272923/default-xmxsize-in-java-8-max-heap-size))
3. ......

#### 1.2.5 方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然 **Java 虚拟机规范把方法区描述为堆的一个逻辑部分**，但是它却有一个别名叫做 **Non-Heap（非堆）**，目的应该是与 Java 堆区分开来。

方法区也被称为永久代。很多人都会分不清方法区和永久代的关系，为此我也查阅了文献。

##### 1.2.5.1 方法区和永久代的关系

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。 **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

##### 1.2.5.2 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

##### 1.2.5.3 为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 呢?

下图来自《深入理解 Java 虚拟机》第 3 版 2.2.5

[![img](https://camo.githubusercontent.com/dcab7d0b9467e806ea394647e4136cc190009a0a406b3604344b60efcfec18d4/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f32303231303432353133343530383131372e706e67)](https://camo.githubusercontent.com/dcab7d0b9467e806ea394647e4136cc190009a0a406b3604344b60efcfec18d4/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f32303231303432353133343530383131372e706e67)

1. 整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

   > 当元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

1. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。
2. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

#### 1.2.6 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误。

> 1. **JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时 hotspot 虚拟机对方法区的实现为永久代**
> 2. **JDK1.7 字符串常量池被从方法区拿到了堆中, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是 hotspot 中的永久代** 。
> 3. **JDK1.8 hotspot 移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace)**

相关问题：JVM 常量池中存储的是对象还是引用呢？： https://www.zhihu.com/question/57109429/answer/151717241 by RednaxelaFX

#### 1.2.7 直接内存

**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）\**与\**缓存区（Buffer）\**的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为\**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

### 1.3 内存划分

**指针碰撞** 

在堆内存中维护一个指针，作为已使用与未使用空间的分界点。在分配内存时，向未使用空间方向移动指针。

优点： 简单高效风险低，在划分内存时只需要移动指针。

缺点：要求内存必须是绝对规整的，即已使用过的内存都在一起。

**空闲列表**

维护一个列表，记录哪些内存块是未分配的。在分配内存时，找到整块足够大的空间划分给对象实例。

优点：不要求内存的连续性。 

缺点：维护成本高，需要单独维护一份列表（越复杂越容易出错）；可能出现内存浪费，虽然还有未分配内存，但都是不连续的，没有足够的整块内存分配给对象实例。

**内存划分并发问题**

1. CAS+失败重试 

   优点：即用即取，不浪费 

   缺点：长时间失败重试会导致性能下降

2. 本地线程分配缓冲 Thread Local Allocation Buffer（TLAB）

   当前线程在创建时，会预先分配内存。本线程内创建对象时，优先在该线程的预分配内存中获取。

   优点：性能最优 缺点：存在浪费

### 1.4 对象访问

**句柄访问**

![image-20211225025528552](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225025528552-0372131.png)

Java 堆中将单独划分一块内存作为句柄池， reference 字 段中存的是对象的句柄地址，局彬中包含了对象实例数据与类型数据各自具体的地址信息。

优点：在对象被移动时（比如发生 G C ）只改变句柄中的实例数据指针即可， reference 不需要变化。

**指针访问**

![image-20211225025618176](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225025618176-0372180.png)

Java 堆中的内存布局就必须考虑如何放置访问类型数据的相关信息， reference 中存储的直接就是对象地址，如果只是访问对象本身的话，就不需要多一次间接访问的开销。

优点：速度更快，节省了一次指针定位的时间开销（ HotSpot用的是指针访问）。



## 2. 垃圾回收

### 2.1 垃圾回收基础理论

线程隔离区域（虚拟机栈、本地方发栈、程序计数器） 如何进行内存管理？

并不是不需要管理，而是这三个区域的内存划分与线程的生命周期保持一致，而且每个栈帧分配的内存大小在类结构确定下来时就是已知的 。

线程共享区域（方法区、堆）如何进行内存管理？

这两个区域有显著的不确定性，一个接口的多个实现类需要的内存可能会不一样，一个方法所执行的不同条件分支所需要的内存也可能不一样，只有运行时才知道。垃圾回收关注的是这部分内存的管理。

#### 2.1.1 如何找到对象

**引用计数法**

原理：对象增加引用时 n+1 ，引用失效时 n‑1

**优点**：**能够直接高效的统计到需要回收的对象，即引用次数为 0 的对象。**

**缺点**： **看似简单，但是实则需要配合大量额外处理才能保证正确性。例如，循环引用问题、异常情况下计数器的正确性补偿问题。（延伸到其他系统设计方案，其实存在一个规律：当使用外部结构来解决当前结构的问题时，同时也要付出维护该外部结构的成本。）**

**可达性分析**

原理：1 . 定义根节点对象；2 . 通过根节点遍历其持有引用的对象；3 . 没有遍历到的标记为待回收对象。

**优点：**  **查找准确度高，不依赖外部结构。**

**缺点：** **遍历效率成为GC的最大性能瓶颈**

hotspot使用可达性分析

**GCRoots**

虚拟机栈中引用的对象（当前栈帧中的参数、局部变量、临时变量等）；方法区中静态属性引用的对象 ；方法区中常量引用的对象 字符串常量池里的引用；本地方法栈中的JNI（本地方法）引用的对象；所有被同步锁（Synchronized关键字）持有的对象；JVM内部的引用（基本数据类型对应的Class对象（Integer、Long、Boolean等），常驻异常对象（NPE、OOM），系统的类加载器（ClassLoader））；反应Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。

**方法区回收**

1. Java堆中不存在该类型的实例 2. 加载该类型的类加载器被回收 3. 该类型的Class对象没有引用

#### 2.1.2 基本的垃圾收集算法

分代收集理论

![image-20211225031136252](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225031136252-0373098.png)



标记清除算法

![image-20211225031404211](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225031404211-0373245.png)

标记复制算法

![image-20211225031429318](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225031429318-0373271.png)

标记整理算法

![image-20211225031520288](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225031520288-0373323.png)



实现细节

![image-20211225031639953](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225031639953.png)



### 2.2 垃圾回收器

演进过程

![jvm垃圾收集器演进](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/jvm垃圾收集器演进.jpg)

**跨代引用解决:**

![image-20211225032112140](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225032112140-0373674.png)

记忆集：记录跨代引用的载体，实现了跨代引用部分扫描；
卡表：一段内存区间，记忆集的存储单元，以较粗的粒度实现记忆集，降低记忆集的空间占用。如果某个卡中存在跨代引用，则称为“脏卡”。

**并发的可达性分析** 

理论前提：全过程必须在一个能保证一致性的稳定快照上完成，所以必须冻结全部用户线程，以保证没有新的对象依赖产生。

![image-20211225032413913](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225032413913-0373856.png)

对象的终态：非黑即白！黑的被保留，白的被回收。
并发的可达性分析中，会出现两个问题：本应回收但未被回收；不应回收却被回收（对象消失问题）。

**对象消失问题**

出现对象消失问题的两个充分必要（同时满足）条件，所以破坏其中一个就可以解决对象消失问题。

1. 赋值器插入了一条或者多条从黑色对象到白色对象的新引用，（扫描之后加入的）；
2. 赋值器删除了全部从灰色对象到该白色对象的直接或者间接引用（扫描未结束就退出的）。

以上的理解：假设A，B，C三个对象。并发标记过程中， A对象已经被扫描完毕，标记为黑，B对象引用了C对象，B对象为灰，C对象为白。在此时（A扫描完毕，B尚未完成扫描前），用户线程建立A到C的引用(条件1)，并删除了B对C的引用(条件2)。这样在并发标记完成后，C对象仍然是白，若回收C对象，则通过A对象调用C时就会出现对象消失问题。

解决方案：

![image-20211225033130774](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225033130774-0374293.png)

CMS采用增量更新，写后屏障（需要在确定关系建立后才能加入），G1采用原始快照，写前屏障（在删除之前记录对应引用）（选择原因 周三答疑问题，待解答）

参考答案: SATB相对增量更新效率会高(当然SATB可能造成更多的浮动垃圾)，因为不需要在重新标记阶段再次深度扫描被删除引用对象，而CMS对增量引用的根对象会做深度扫描，G1因为很多对象都位于不同的region，CMS就一块老年代区域，重新深度扫描对象的话G1的代价会比CMS高，所以G1选择SATB不深度扫描对象，只是简单标记，等到下一轮GC再深度扫描。

**以上未能理解**

CMS

![image-20211225033640361](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225033640361-0374601.png)

缺点
对资源敏感：单核或者伪多核的机器上对用户线程的影响较大；
无法处理“浮动垃圾”：浮动垃圾过多时，易触发Concurrent Mode Failure，导致启用Serial进行一次STW的FullGC；
碎片较多：由于使用的回收算法是“标记-清除”算法，会导致出现过多的内存碎片。

G1

在G1收集器中，依然存在老年代与新生代的概念，但是其内存区域不是固定的，都是一系列Region的动态集合。
在进行回收时，G1收集器会评估每个Region的回收价值，形成一个排序，然后根据JVM的收集时间上限，框定出本次需要回收Region。

![image-20211225033757102](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225033757102-0374679.png)

G1垃圾回收循环

![image-20211225034313233](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225034313233-0374995.png)

> 1. Young-only phase: This phase starts with a few Normal young collections that promote objects into the old generation. The transition between the young-only phase and the space-reclamation phase starts when the old generation occupancy reaches a certain threshold, the Initiating Heap Occupancy threshold. At this time, G1 schedules a Concurrent Start young collection instead of a Normal young collection. 
>    - Concurrent Start : This type of collection starts the marking process in addition to performing a Normal young collection. Concurrent marking determines all currently reachable (live) objects in the old generation regions to be kept for the following space-reclamation phase. While collection marking hasn’t completely finished, Normal young collections may occur. Marking finishes with two special stop-the-world pauses: Remark and Cleanup. 
>    - Remark: This pause finalizes the marking itself, performs global reference processing and class unloading, reclaims completely empty regions and cleans up internal data structures. Between Remark and Cleanup G1 calculates information to later be able to reclaim free space in selected old generation regions concurrently, which will be finalized in the Cleanup pause.
>    - Cleanup: This pause determines whether a space-reclamation phase will actually follow. If a space-reclamation phase follows, the young-only phase completes with a single Prepare Mixed young collection. 
> 2. Space-reclamation phase: This phase consists of multiple Mixed collections that in addition to young generation regions, also evacuate live objects of sets of old generation regions. The space-reclamation phase ends when G1 determines that evacuating more old generation regions wouldn't yield enough free space worth the effort.(The phase ends when the remaining amount of space that can be reclaimed in the collection set candidate regions is less than the percentage set by` -XX:G1HeapWastePercent`.)

G1内存区域划分

![image-20211225033841815](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225033841815-0374723.png)

大对象Humongous 

1. 需要一段连续的regions，其占据的最后一个region的剩余区域在该对象被回收前都是不可用的。
2. 大对象只有在clean up阶段或者Full GC阶段才能被回收。基本类型数组除外，此类型大对象会在任何垃圾回收停顿中被尝试回收，此行为可通过-XX:-G1EagerReclaimHumongousObjects 控制，默认开启。
3. 导致垃圾回收过早发生 每次分配大对象都会检测IHOP
4. 大对象永不移动。

shenandoah

相比G1的优化： 链接矩阵，转发指针(读屏障，影响性能)

![image-20211225040533974](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225040533974-0376336.png)

ZGC

读屏障、染色指针和内存多重映射 可并发的标记-整理算法。

![image-20211225040656644](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.12.26/image-20211225040656644-0376419.png)