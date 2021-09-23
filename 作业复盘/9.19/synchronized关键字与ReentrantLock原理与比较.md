# Java 并发进阶常见面试题总结

## 1.synchronized 关键字

### 1.1.synchronized 关键字简介

**`synchronized` 关键字解决的是多个线程之间访问资源的同步性，`synchronized`关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。**

在 Java 早期版本中，`synchronized` 属于 **重量级锁**，效率低下。

原因：

因为监视器锁（monitor）是依赖于底层的操作系统的 `Mutex Lock` 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

而在JDK1.6 之后对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

所以，目前不论是各种开源框架还是 JDK 源码都大量使用了 `synchronized` 关键字。

### 1.2. synchronized 关键字的使用

**synchronized 关键字最主要的三种使用方式：**

**1.修饰实例方法:** 作用于当前对象实例加锁，进入同步代码前要获得 **当前对象实例的锁**

```java
synchronized void method() {    //业务代码}
```

**2.修饰静态方法:** 也就是给当前类加锁，会作用于类的所有对象实例 ，进入同步代码前要获得 **当前 class 的锁**。因为静态成员不属于任何一个实例对象，是类成员（ *static 表明这是该类的一个静态资源，不管 new 了多少个对象，只有一份*）。所以，如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，**因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁**。

```java
synchronized static void method() {    //业务代码}
```

**3.修饰代码块** ：指定加锁对象，对给定对象/类加锁。`synchronized(this|object)` 表示进入同步代码库前要获得**给定对象的锁**。`synchronized(类.class)` 表示进入同步代码前要获得 **当前 class 的锁**

```java
synchronized(this) {    //业务代码}
```

**总结：**

- `synchronized` 关键字加到 `static` 静态方法和 `synchronized(class)` 代码块上都是是给 Class 类上锁。
- `synchronized` 关键字加到实例方法上是给对象实例上锁。
- 尽量不要使用 `synchronized(String a)` 因为 JVM 中，字符串常量池具有缓存功能！

 `synchronized` 关键字的具体使用：

**双重校验锁实现对象单例（线程安全）**

```java
public class Singleton {    
  private volatile static Singleton uniqueInstance;    
  private Singleton() {    }    
  public  static Singleton getUniqueInstance() {       
    //先判断对象是否已经实例过，没有实例化过才进入加锁代码        
    if (uniqueInstance == null) {            //类对象加锁            
      synchronized (Singleton.class) {                
        if (uniqueInstance == null) {                    
          uniqueInstance = new Singleton();                
        }            
      }       
    }        
    return uniqueInstance;   
  }
}
```

另外，需要注意 `uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要。

`uniqueInstance` 采用 `volatile` 关键字修饰也是很有必要的， `uniqueInstance = new Singleton();` 这段代码其实是分为三步执行：

1. 为 `uniqueInstance` 分配内存空间
2. 初始化 `uniqueInstance`
3. 将 `uniqueInstance` 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getUniqueInstance`() 后发现 `uniqueInstance` 不为空，因此返回 `uniqueInstance`，但此时 `uniqueInstance` 还未被初始化。

使用 `volatile` 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

此外，**构造方法不能使用 synchronized 关键字修饰。**构造方法本身就属于线程安全的，不存在同步的构造方法一说。

### 1.3. synchronized 关键字的底层原理

#### 1.3.1. jvm

**synchronized 关键字底层原理属于 JVM 层面。**

JVM基于进入和退出Monitor对象来实现方法同步和代码块同步, 但是两者的实现细节不一样.

1. 代码块同步: 通过使用monitorenter和monitorexit指令实现的.
2. 同步方法: ACC_SYNCHRONIZED修饰

monitorenter指令是在编译后插入到同步代码块的开始位置, 而monitorexit指令是在编译后插入到同步代码块的结束处或异常处，异常处monitorexit后面会追加goto  line 指令 跳转到对应执行行数。

**两者的本质都是对对象监视器 monitor 的获取。**

#### 1.3.2.锁升级

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

![img](https://raw.githubusercontent.com/1026359484n/lagou_private_education/main/作业复盘/9.19/v2-770c0db5330d3ca0a4e4228205a796ef_1440w.jpg)

##### 1.3.2.2.无锁状态

jvm在启动时默认打开了偏向锁(jdk11),此时对象处于无锁可偏向状态，线程进入临界区会升级为偏向锁。除了无锁可偏向状态，还有无锁不可偏向状态，如果修改jvm启动参数-XX:-UseBiasedLocking 禁止用偏向锁,调用hashcode或者锁从轻量级锁或重量级锁变回无锁状态，会变为无锁不可偏向，流程如下:

![img](https://raw.githubusercontent.com/1026359484n/lagou_private_education/main/作业复盘/9.19/v2-dffc714665b45eafa0e11d36e315bec3_1440w.jpg)

##### 无锁可偏向状态下，如果调用系统的`hashCode()`方法，会在`markword`中写入`hashCode`，并修改可偏向状态标志位。在这个状态下，如果有线程尝试获取锁，会直接从无锁升级到轻量级锁，不会再升级为偏向锁。

##### 1.3.2.5.偏向锁

偏向锁是针对于一个线程而言的, 线程获得锁之后就不会再有解锁等操作了, 这样可以省略很多开销. 假如有第二个线程进入临界区, 进而升级成轻量级锁了。其实在大部分情况下, 都会是同一个线程进入同一块同步代码块的. 这也是为什么会有偏向锁出现的原因。

**加锁**：当一个线程访问同步块并获取锁时, 会在锁对象的对象头和栈帧中的锁记录里存储锁偏向的线程ID, 以后该线程进入和退出同步块时不需要进行CAS操作来加锁和解锁, 只需要简单的测试一下锁对象的对象头的MarkWord里是否存储着指向当前线程的偏向锁(线程ID是当前线程), 如果测试成功, 表示线程已经获得了锁，**接释放流程**; 如果测试失败, 则需要再测试一下MarkWord中偏向锁的标识是否设置成1(表示当前是偏向锁), 如果设置了, 则尝试使用CAS将锁对象的对象头的偏向锁指向当前线程，如果成功，**接释放流程**。如果没有设置或者cas设置失败, 则尝试加轻量级锁。

**释放**:  判断是偏向锁时，检查对象头Mark Word中记录的Thread Id是否是当前线程ID，如果是，则表明当前线程已经获得对象锁，以后该线程进入同步块时，不需要CAS进行加锁，只会往当前线程的栈中添加一条Displaced Mark Word（用于存储锁对象目前的Mark Word的拷贝）为空的Lock Record中，用来统计重入的次数。退出同步块释放偏向锁时，则依次删除对应Lock Record，但是不会修改对象头中的Thread Id。

**撤销**： 偏向锁撤销是指在获取偏向锁的过程中因不满足条件导致要将锁对象改为非偏向锁状态。偏向锁的撤销需要等待全局安全点（safe point，代表了一个状态，在该状态下所有线程都是暂停的）,暂停持有偏向锁的线程，检查持有偏向锁的线程状态（遍历当前JVM的所有线程，如果能找到，则说明偏向的线程还存活），如果线程还存活，则检查线程是否在执行同步代码块中的代码，如果是，则升级为轻量级锁，进行CAS竞争锁。

场景: 适用于只有一个线程进入临界区的情况

##### 1.3.2.3.轻量级锁

加锁：通过CAS尝试将Mark Word更新为指向BasicLock对象的指针，如果更新成功，表示竞争到锁，则执行同步代码，如果是重入，则设置Displaced Mark Word为null。如果有多个线程竞争轻量级锁，轻量级锁需要膨胀升级为重量级锁。

解锁：cas替换`mark word`。若解锁时有竞争，会调用`inflate`方法进行重量级锁膨胀，升级到到重量级锁后再执行`exit`方法。

场景：多个线程不同时段访问临界区，此时不需要阻塞操作，连monitor对象都不需要了。

##### 1.3.2.4.重量级锁

cas设置owner，自适应自旋，多次尝试，成功则执行同步代码，失败则挂起。

![640-20210920021850416](https://raw.githubusercontent.com/1026359484n/lagou_private_education/main/作业复盘/9.19/640-20210920021850416.webp)

场景：多个线程同时访问临界区。

##### 1.3.2.5.总结

![image-20210920004004549](https://raw.githubusercontent.com/1026359484n/lagou_private_education/main/作业复盘/9.19/image-20210920004004549.png)

**一旦发生锁竞争或者调用wait()/notify()方法，就会升级为重量级锁，多个线程轮流顺利进入临界区，不叫锁竞争，是轻量级锁。始终只有一个线程进入临界区才有可能是偏向锁**

## 6. AQS

### 6.1. AQS 介绍

AQS 的全称为（`AbstractQueuedSynchronizer`），这个类在`java.util.concurrent.locks`包下面。

AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask` 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

### 6.2. AQS 原理分析

AQS 原理这部分参考了部分博客，在 5.2 节末尾放了链接。

> 在面试中被问到并发知识的时候，大多都会被问到“请你说一下自己对于 AQS 原理的理解”。下面给大家一个示例供大家参加，面试不是背题，大家一定要加入自己的思想，即使加入不了自己的思想也要保证自己能够通俗的讲出来而不是背出来。

下面大部分内容其实在 AQS 类注释上已经给出了，不过是英语看着比较吃力一点，感兴趣的话可以看看源码。

#### 6.2.1. AQS 原理概览

**AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

看个 AQS(AbstractQueuedSynchronizer)原理图：

[![AQS原理图](https://raw.githubusercontent.com/1026359484n/lagou_private_education/main/作业复盘/9.19/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f4151532545352538452539462545372539302538362545352539422542452e706e67.png)](https://camo.githubusercontent.com/ac2a92061ff7f1176d98be55181fe6615a7137bab4337540f96fc5c2056ba652/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f4151532545352538452539462545372539302538362545352539422542452e706e67)

AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

```
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

状态信息通过 protected 类型的 getState，setState，compareAndSetState 进行操作

```java
//返回同步状态的当前值
protected final int getState() {
  return state;
}
//设置同步状态的值
protected final void setState(int newState) {   
  state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {    
  return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

#### 6.2.2. AQS 对资源的共享方式

**AQS 定义两种资源共享方式**

- Exclusive（独占）：只有一个线程能执行，如`ReentrantLock`。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- **Share**（共享）：多个线程可同时执行，如` CountDownLatch`、`Semaphore`、 `CyclicBarrier`、`ReadWriteLock` 我们都会在后面讲到。

`ReentrantReadWriteLock` 可以看成是组合式，因为 `ReentrantReadWriteLock` 也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在顶层实现好了。

#### 6.2.3. AQS 底层使用了模板方法模式

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承 `AbstractQueuedSynchronizer` 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

**AQS 使用了模板方法模式，自定义同步器时需要重写下面几个 AQS 提供的模板方法：**

```
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 `UnsupportedOperationException`。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS 类中的其他方法都是 final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多少次，这样才能保证 state 是能回到零态的。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。

## 3. 谈谈 synchronized 和 ReentrantLock 的区别

#### 3.1. 两者都是可重入锁

**“可重入锁”** 指的是自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增 1，所以要等到锁的计数器下降为 0 时才能释放锁。

#### 3.2.synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API

`synchronized` 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 `synchronized` 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。`ReentrantLock` 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

#### 3.3.ReentrantLock 比 synchronized 增加了一些高级功能

相比`synchronized`，`ReentrantLock`增加了一些高级功能。主要来说主要有三点：

- **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来制定是否是公平的。
- **可实现选择性通知（锁可以绑定多个条件）**: `synchronized`关键字与`wait()`和`notify()`/`notifyAll()`方法相结合可以实现等待/通知机制。`ReentrantLock`类当然也可以实现，但是需要借助于`Condition`接口与`newCondition()`方法。

> `Condition`是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个`Lock`对象中可以创建多个`Condition`实例（即对象监视器），**线程对象可以注册在指定的`Condition`中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用`notify()/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，用`ReentrantLock`类结合`Condition`实例可以实现“选择性通知”** ，这个功能非常重要，而且是 Condition 接口默认提供的。而`synchronized`关键字就相当于整个 Lock 对象中只有一个`Condition`实例，所有的线程都注册在它一个身上。如果执行`notifyAll()`方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而`Condition`实例的`signalAll()`方法 只会唤醒注册在该`Condition`实例中的所有等待线程。

