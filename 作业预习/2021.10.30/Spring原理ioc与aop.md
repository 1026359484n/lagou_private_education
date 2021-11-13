# **Spring** 原理 ioc与aop



## 1. 核⼼思想

IOC和AOP不是spring提出的，在spring之前就已经存在，只不过更偏向于理论化，spring在技术层次把这两个思想做了⾮常好的实现（Java） 

### **1.1 IoC**？

IoC Inversion of Control (控制反转/反转控制)，它是⼀个技术思想，不是⼀个技术实现。它描述的是Java开发领域对象的创建，管理的问题。

传统开发⽅式：⽐如类A依赖于类B，往往会在类A中new⼀个B的对象IoC思想下开发⽅式：我们不⽤⾃⼰去new对象了，⽽是由IoC容器（Spring框架）去帮助我们实例化对象并且管理它，我们需要使⽤哪个对象，去问IoC容器要即可

我们丧失了创建、管理对象的权利,却不⽤再考虑对象的创建、管理等⼀系列事情了。

控制反转中**控制**指的是对象创建（实例化、管理）的权利,**反转**是指控制权交给外部环境了（spring框架、IoC容器）

**IoC**解决对象之间的耦合问题**1.3 IoC**和**DI**的区别

DI：Dependancy Injection（依赖注⼊）怎么理解：IOC和DI描述的是同⼀件事情，只不过⻆度不⼀样罢了

### **1.2 AOP**

AOP: Aspect oriented Programming ⾯向切⾯编程/⾯向⽅⾯编程。AOP是OOP的延续，从OOP说起OOP三⼤特征：封装、继承和多态。

oop是⼀种垂直继承体系，OOP编程思想可以解决⼤多数的代码重复问题，但是有⼀些情况是处理不了的，比如多个⽅法中相同位置出现了重复代码，OOP就解决不了横切逻辑代码。

横切逻辑代码存在横切代码重复的问题，横切逻辑代码和业务代码混杂在⼀起，代码臃肿，维护不⽅便。而AOP独辟蹊径提出横向抽取机制，将横切逻辑代码和业务逻辑代码拆分。代码拆分容易，那么如何在不改变原有业务逻辑的情况下，悄⽆声息的把横切逻辑代码应⽤到原有的业务逻辑中，达到和原来⼀样的效果，这个是⽐较难的。

Spring aop解决的就是在不改变原有业务逻辑情况下，增强横切逻辑代码，根本上解耦合，避免横切逻辑代码重复

为什么叫做⾯向切⾯编程

「切」：指的是横切逻辑，原有业务逻辑代码我们不能动，只能操作横切逻辑代码，所以⾯向横切逻辑

「⾯」：横切逻辑代码往往要影响的是很多个⽅法，每⼀个⽅法都如同⼀个点，多个点构成⾯，有⼀个⾯的概念在⾥⾯



## 2. Spring IOC应⽤

### **2.1 Spring IoC基础**

#### BeanFactory与**ApplicationContext**区别

BeanFactory是Spring框架中IoC容器的顶层接⼝,它只是⽤来定义⼀些基础功能,定义⼀些基础规范,⽽ApplicationContext是它的⼀个⼦接⼝，所以ApplicationContext是具备BeanFactory提供的全部功能的。

通常，我们称BeanFactory为SpringIOC的基础容器，ApplicationContext是容器的⾼级接⼝，⽐BeanFactory要拥有更多的功能，⽐如说国际化⽀持和资源访问（xml，java配置类）等等启动 IoC 容器的⽅式

Java环境下启动IoC容器

ClassPathXmlApplicationContext：从类的根路径下加载配置⽂件（推荐使⽤）

FileSystemXmlApplicationContext：从磁盘路径上加载配置⽂件

AnnotationConfifigApplicationContext：纯注解模式下启动Spring容器

#### **Bean的范围及⽣命周期**

- 作⽤范围的改变

  在spring框架管理Bean对象的创建时，Bean对象默认都是单例的，但是它⽀持配置的⽅式改变作⽤范围。作⽤范围官⽅提供的说明如下图：

  ![image-20211030200432364](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.30/image-20211030200432364-5595474.png)

  在上图中提供的这些选项中，我们实际开发中⽤到最多的作⽤范围就是singleton（单例模式)和prototype（原型模式，也叫多例模式）。配置⽅式参考下⾯的代码：

  ```xml
  <!--配置service对象-->
  <bean id="transferService" class="com.lagou.service.impl.TransferServiceImpl" scope="singleton">
  </bean>
  ```

- 不同作⽤范围的⽣命周期

  单例模式：**singleton**

  对象出⽣：当创建容器时，对象就被创建了。

  对象活着：只要容器在，对象⼀直活着。

  对象死亡：当销毁容器时，对象就被销毁了。

  ⼀句话总结：单例模式的bean对象⽣命周期与容器相同。

  多例模式：**prototype**

  对象出⽣：当使⽤对象时，创建新的对象实例。

  对象活着：只要对象在使⽤中，就⼀直活着。

  对象死亡：当对象⻓时间不⽤时，被java的垃圾回收器回收了。

  ⼀句话总结：多例模式的bean对象，spring框架只负责创建，不负责销毁。

### **2.2 Spring IoC容器初始化主体流程**

**2.2.1 Spring IoC**的容器体系

IoC容器是Spring的核⼼模块，是抽象了对象管理、依赖关系管理的框架解决⽅案。Spring 提供了很多的容器，其中 BeanFactory 是顶层容器（根容器），不能被实例化，它定义了所有 IoC 容器 必须遵从的⼀套原则，具体的容器实现可以增加额外的功能，⽐如我们常⽤到的ApplicationContext，其下更具体的实现如 ClassPathXmlApplicationContext 包含了解析 xml 等⼀系列的内容，

AnnotationConfifigApplicationContext 则是包含了注解解析等⼀系列的内容。Spring IoC 容器继承体系⾮常聪明，需要使⽤哪个层次⽤哪个层次即可，不必使⽤功能⼤⽽全的。

BeanFactory 顶级接⼝⽅法栈如下

BeanFactory 容器继承体系通过其接⼝设计，我们可以看到我们⼀贯使⽤的 ApplicationContext 除了继承BeanFactory的⼦接⼝，还继承了ResourceLoader、MessageSource等接⼝，因此其提供的功能也就更丰富了。下⾯我们以 ClasspathXmlApplicationContext 为例，深⼊源码说明 IoC 容器的初始化流程。

**1.2 Bean**⽣命周期关键时机点

 Bean对象创建的⼏个关键时机点代码层级的调⽤都在AbstractApplicationContext 类 的 refresh ⽅法中，可⻅这个⽅法对于Spring IoC 容器初始化来说相当关键，汇总如下：

![image-20211030214705413](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.30/image-20211030214705413-5601627.png)

**1.3 Spring IoC**容器初始化主流程

Spring IoC 容器初始化的关键环节在 AbstractApplicationContext#refresh() ⽅法中，我们查看 refresh ⽅法来俯瞰容器创建的主体流。

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
  synchronized (this.startupShutdownMonitor) {
    // 第⼀步：刷新前的预处理
    prepareRefresh();
    /*
    第⼆步：
    获取BeanFactory；默认实现是DefaultListableBeanFactory
    加载BeanDefition 并注册到 BeanDefitionRegistry
    */
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
    // 第三步：BeanFactory的预准备⼯作（BeanFactory进⾏⼀些设置，⽐如context的类加载器等）
    prepareBeanFactory(beanFactory);
    try {
      // 第四步：BeanFactory准备⼯作完成后进⾏的后置处理⼯作
      postProcessBeanFactory(beanFactory);
      // 第五步：实例化并调⽤实现了BeanFactoryPostProcessor接⼝的Bean
      invokeBeanFactoryPostProcessors(beanFactory);
      // 第六步：注册BeanPostProcessor（Bean的后置处理器），在创建bean的前后等执⾏
      registerBeanPostProcessors(beanFactory);
      // 第七步：初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
      initMessageSource();
      // 第⼋步：初始化事件派发器
      initApplicationEventMulticaster();
      // 第九步：⼦类重写这个⽅法，在容器刷新的时候可以⾃定义逻辑
      onRefresh();
      // 第⼗步：注册应⽤的监听器。就是注册实现了ApplicationListener接⼝的监听器bean
      registerListeners();
      /*
      第⼗⼀步：
      初始化所有剩下的⾮懒加载的单例bean
      初始化创建⾮懒加载⽅式的单例Bean实例（未设置属性）
      填充属性
      初始化⽅法调⽤（⽐如调⽤afterPropertiesSet⽅法、init-method⽅法）
      调⽤BeanPostProcessor（后置处理器）对实例bean进⾏后置处
      */
      finishBeanFactoryInitialization(beanFactory);
      /*
      第⼗⼆步：
      完成context的刷新。主要是调⽤LifecycleProcessor的onRefresh()⽅法，并且发布事件 （ContextRefreshedEvent）
      */
      finishRefresh();
    }
  }
} 
```

第**2**节 **BeanFactory**创建流程

**2.1** 获取**BeanFactory**⼦流程

时序图如下

![image-20211030202554496](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.30/image-20211030202554496.png)

**2.2 BeanDefinition**加载解析及注册⼦流程

（**1**）该⼦流程涉及到如下⼏个关键步骤

​		**Resource**定位：指对BeanDefifinition的资源定位过程。通俗讲就是找到定义Javabean信息的XML⽂件，并将其封装成Resource对象。

​		**BeanDefifinition**载⼊ ：把⽤户定义好的Javabean表示为IoC容器内部的数据结构，这个容器内部的数据结构就是BeanDefifinition。

​		注册**BeanDefifinition**到 **IoC** 容器

### 2.3 Bean创建流程

![image-20211030202049111](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.30/image-20211030202049111-5596450.png)

### **2.4 lazy-init 延迟加载机制原理**

lazy-init 延迟加载机制分析

普通 Bean 的初始化是在容器启动初始化阶段执⾏的，⽽被lazy-init=true修饰的 bean 则是在从容器⾥第⼀次进⾏context.getBean() 时进⾏触发。Spring 启动的时候会把所有bean信息(包括XML和注解)解析转化成Spring能够识别的BeanDefifinition并存到Hashmap⾥供下⾯的初始化时⽤，然后对每个BeanDefifinition 进⾏处理，如果是懒加载的则在容器初始化阶段不处理，其他的则在容器初始化阶段进⾏初始化并依赖注⼊。

```java
public void preInstantiateSingletons() throws BeansException {
  // 所有beanDefinition集合
  List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
  // 触发所有⾮懒加载单例bean的初始化
  for (String beanName : beanNames) {
    // 获取bean 定义
    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
    // 判断是否是懒加载单例bean，如果是单例的并且不是懒加载的则在容器创建时初始化
    if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
      // 判断是否是 FactoryBean
      if (isFactoryBean(beanName)) {
        final FactoryBean<?> factory = (FactoryBean<?>)getBean(FACTORY_BEAN_PREFIX + beanName);
        boolean isEagerInit;
        if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
          isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
            @Override
            public Boolean run() {
              return ((SmartFactoryBean<?>) factory).isEagerInit();
            }
          }, getAccessControlContext());
        }
      }else {
        /*
        如果是普通bean则进⾏初始化并依赖注⼊，此 getBean(beanName)接下来触发的逻辑和懒加载时 context.getBean("beanName") 所触发的逻辑是⼀样的
        */
        getBean(beanName);
      }
    }
  }
}
```

总结

对于被修饰为lazy-init的bean Spring 容器初始化阶段不会进⾏ init 并且依赖注⼊，当第⼀次进⾏getBean时候才进⾏初始化并依赖注⼊

对于⾮懒加载的bean，getBean的时候会从缓存⾥头获取，因为容器初始化阶段 Bean 已经初始化完成并缓存了起来

### **2.5 Spring IoC循环依赖问题**

#### **2.5.1** 什么是循环依赖

循环依赖其实就是循环引⽤，也就是两个或者两个以上的 Bean 互相持有对⽅，最终形成闭环。⽐如A依赖于B，B依赖于C，C⼜依赖于A。注意，这⾥不是函数的循环调⽤，是对象的相互依赖关系。循环调⽤其实就是⼀个死循环，除⾮有终结条件。

Spring中循环依赖场景有：构造器的循环依赖（构造器注⼊）、Field 属性的循环依赖（set注⼊）

其中，~~构造器的循环依赖问题在⽆法解决，只能拋出 BeanCurrentlyInCreationException 异常~~，在解决属性循环依赖时，spring采⽤的是提前暴露对象的⽅法。

#### **2.5.2** 循环依赖处理机制

单例 bean 构造器参数循环依赖（~~⽆法解决~~ 懒加载@Lazy）

prototype 原型 bean循环依赖（⽆法解决）

对于原型bean的初始化过程中不论是通过构造器参数循环依赖还是通过setXxx⽅法产⽣循环依赖，Spring都 会直接报错处理。

AbstractBeanFactory.doGetBean()⽅法：

```java
if (isPrototypeCurrentlyInCreation(beanName)) {

throw new BeanCurrentlyInCreationException(beanName);

}

protected boolean isPrototypeCurrentlyInCreation(String beanName) {

Object curVal = this.prototypesCurrentlyInCreation.get();

return (curVal != null &&

 (curVal.equals(beanName) || (curVal instanceof Set && ((Set<?>)

curVal).contains(beanName))));

}
```

在获取bean之前如果这个原型bean正在被创建则直接抛出异常。原型bean在创建之前会进⾏标记这个beanName正在被创建，等创建结束之后会删除标记

```java
try {
  //创建原型bean之前添加标记
  beforePrototypeCreation(beanName);
  //创建原型bean
  prototypeInstance = createBean(beanName, mbd, args);
}finally {
  //创建原型bean之后删除标记
  afterPrototypeCreation(beanName);
}
```

总结：Spring 不⽀持原型 bean 的循环依赖。

单例bean通过setXxx或者@Autowired进⾏循环依赖Spring 的循环依赖的理论依据基于 Java 的引⽤传递，当获得对象的引⽤时，对象的属性是可以延后设置的，但是构造器必须是在获取引⽤之前Spring通过setXxx或者@Autowired⽅法解决循环依赖其实是通过提前暴露⼀个ObjectFactory对象来完成的，简单来说ClassA在调⽤构造器完成对象初始化之后，在调⽤ClassA的setClassB⽅法之前就把ClassA实例化的对象通过ObjectFactory提前暴露到Spring容器中。

Spring容器初始化ClassA通过构造器初始化对象后提前暴露到Spring容器。

```java
boolean earlySingletonExposure = (mbd.isSingleton() &&
                                  this.allowCircularReferences &&
                                  isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
  if (logger.isDebugEnabled()) {
    logger.debug("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
}

//将初始化后的对象提前已ObjectFactory对象注⼊到容器中

addSingletonFactory(beanName, new ObjectFactory<Object>() {
  @Override
  public Object getObject() throws BeansException {
    return getEarlyBeanReference(beanName, mbd, bean);
  }
});
}
```

ClassA调⽤setClassB⽅法，Spring⾸先尝试从容器中获取ClassB，此时ClassB不存在Spring容器中。

Spring容器初始化ClassB，同时也会将ClassB提前暴露到Spring容器中ClassB调⽤setClassA⽅法，Spring从容器中获取ClassA ，因为第⼀步中已经提前暴露了ClassA，因此可以获取到ClassA实例ClassA通过spring容器获取到ClassB，完成了对象初始化操作。

这样ClassA和ClassB都完成了对象初始化操作，解决了循环依赖问题。

**Spring 需要三级缓存的目的是为了在没有循环依赖的情况下，延迟代理对象的创建，使 Bean 的创建符合 Spring 的设计原则**

## 3. Spring AOP应⽤

AOP本质：在不改变原有业务逻辑的情况下增强横切逻辑，横切逻辑代码往往是权限校验代码、⽇志代码、事务控制代码、性能监控代码。

### **3.1 AOP 相关术语**

#### **3.1.1 业务主线**

在讲解AOP术语之前，我们先来看⼀下下⾯这两张图，它们就是第三部分案例需求的扩展（针对这些扩展的需求，我们只进⾏分析，在此基础上去进⼀步回顾AOP，不进⾏实现）上图描述的就是未采⽤AOP思想设计的程序，当我们红⾊框中圈定的⽅法时，会带来⼤量的重复劳动。

程序中充斥着⼤量的重复代码，使我们程序的独⽴性很差。⽽下图中是采⽤了AOP思想设计的程序，它把红框部分的代码抽取出来的同时，运⽤动态代理技术，在运⾏期对需要使⽤的业务逻辑⽅法进⾏增强。

**3.1.2 AOP** 术语名词 解释

**Joinpoint(**连接点**)**

它指的是那些可以⽤于把增强代码加⼊到业务主线中的点，那么由上图中我们可以看出，这些点指的就是⽅法。在⽅法执⾏的前后通过动态代理技术加⼊增强的代码。在Spring框架AOP思想的技术实现中，也只⽀持⽅法类型的连接点。

**Pointcut(**切⼊点**)**

它指的是那些已经把增强代码加⼊到业务主线进来之后的连接点。由上图中，我们看出表现层 transfer ⽅法就只是连接点，因为判断访问权限的功能并没有对其增强。

**Advice(**通知**/**增强**)**

它指的是切⾯类中⽤于提供增强功能的⽅法。并且不同的⽅法增强的时机是不⼀样的。⽐如，开启事务肯定要在业务⽅法执⾏之前执⾏；提交事务要在业务⽅法正常执⾏之后执⾏，⽽回滚事务要在业务⽅法执⾏产⽣异常之后执⾏等等。那么这些就是通知的类型。其分类有：前置通知 后置通知 异常通知 最终通知 环绕通知。

**Target(**⽬标对象**)** 

它指的是代理的⽬标对象。即被代理对象。

**Proxy(**代理**)** 它指的是⼀个类被AOP织⼊增强后，产⽣的代理类。即代理对象。

**Weaving(**织 ⼊**)**

它指的是把增强应⽤到⽬标对象来创建新的代理对象的过程。spring采⽤动态代理织⼊，⽽AspectJ采⽤编译期织⼊和类装载期织⼊。

**Aspect(**切 ⾯**)**

它指定是增强的代码所关注的⽅⾯，把这些相关的增强代码定义到⼀个类中，这个类就是切⾯类。例如，事务切⾯，它⾥⾯定义的⽅法就是和事务相关的，像开启事务，提交事务，回滚事务等等，不会定义其他与事务⽆关的⽅法。我们前⾯的案例中 TrasnactionManager 就是⼀个切⾯。

连接点：⽅法开始时、结束时、正常运⾏完毕时、⽅法异常时等这些特殊的时机点，我们称之为连接点，项⽬中每个⽅法都有连接点，连接点是⼀种候选点

切⼊点：指定AOP思想想要影响的具体⽅法是哪些，描述感兴趣的⽅法

Advice增强：

第⼀个层次：指的是横切逻辑

第⼆个层次：⽅位点（在某⼀些连接点上加⼊横切逻辑，那么这些连接点就叫做⽅位点，描述的是具体

的特殊时机）

Aspect切⾯：切⾯概念是对上述概念的⼀个综合

Aspect切⾯= 切⼊点+增强 = 切⼊点（锁定⽅法） + ⽅位点（锁定⽅法中的特殊时机）+ 横切逻辑

众多的概念，⽬的就是为了锁定要在哪个地⽅插⼊什么横切逻辑代码第**2**节 **Spring**中**AOP**的代理选择

Spring 实现AOP思想使⽤的是动态代理技术

默认情况下，Spring会根据被代理对象是否实现接⼝来选择使⽤JDK还是CGLIB。当被代理对象没有实现任何接⼝时，Spring会选择CGLIB。当被代理对象实现了接⼝，Spring会选择JDK官⽅的代理技术，不过我们可以通过配置的⽅式，让Spring强制使⽤CGLIB。 

### 3.3 **Spring**中**AOP**的配置⽅式

在Spring的AOP配置中，也和IoC配置⼀样，⽀持3类配置⽅式。

第⼀类：使⽤XML配置

第⼆类：使⽤XML+注解组合配置

第三类：使⽤纯注解配置

### **3.4 Spring中AOP实现**

#### 3.4.1 切入点表达式

- 关于切⼊点表达式

  上述配置实现了对 TransferServiceImpl 的 updateAccountByCardNo ⽅法进⾏增强，在其执⾏之前，输出了记录⽇志的语句。这⾥⾯，我们接触了⼀个⽐较陌⽣的名称：切⼊点表达式，它是做什么的呢？我们往下看。

- 概念及作⽤

  切⼊点表达式，也称之为AspectJ切⼊点表达式，指的是遵循特定语法结构的字符串，其作⽤是⽤于对符合语法格式的连接点进⾏增强。它是AspectJ表达式的⼀部分。

- 关于AspectJ

  AspectJ是⼀个基于Java语⾔的AOP框架，Spring框架从2.0版本之后集成了AspectJ框架中切⼊点表达式的部分，开始⽀持AspectJ切⼊点表达式。

- 切⼊点表达式使⽤示例

  ```
  全限定⽅法名 访问修饰符 返回值 包名.包名.包名.类名.⽅法名(参数列表)
  
  全匹配⽅式：
  
  public void com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
  
  访问修饰符可以省略
  void com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
  
  返回值可以使⽤*，表示任意返回值
  * com.lagou.service.impl.TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
  包名可以使⽤.表示任意包，但是有⼏级包，必须写⼏个*
  ....TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
  
  包名可以使⽤..表示当前包及其⼦包
  
  * ..TransferServiceImpl.updateAccountByCardNo(com.lagou.pojo.Account)
  
  类名和⽅法名，都可以使⽤.表示任意类，任意⽅法
  * ...(com.lagou.pojo.Account)
  
  参数列表，可以使⽤具体类型
  
  基本类型直接写类型名称：
  int
  
  引⽤类型必须写全限定类名：
  java.lang.String
  
  参数列表可以使⽤*，表示任意参数类型，但是必须有参数
  \* *..*.*(*)
  
  参数列表可以使⽤..，表示有⽆参数均可。有参数可以是任意类型
  \* *..*.*(..)
  
  全通配⽅式：
  \* *..*.*(..)
  ```



#### **3.4.2 XML+**注解模式

XML 中开启 Spring 对注解 AOP 的⽀持

```xml
<!--开启spring对注解aop的⽀持-->
<aop:aspectj-autoproxy/
```

示例

```java
/**
* 模拟记录⽇志
*/

@Component
@Aspect
public class LogUtil {
  /**
  * 我们在xml中已经使⽤了通⽤切⼊点表达式，供多个切⾯使⽤，那么在注解中如何使⽤呢？* 第⼀步：编写⼀个⽅法
  * 第⼆步：在⽅法使⽤@Pointcut注解
  * 第三步：给注解的value属性提供切⼊点表达式
  * 细节：
  * 1.在引⽤切⼊点表达式时，必须是⽅法名+()，例如"pointcut()"。 
  * 2.在当前切⾯中使⽤，可以直接写⽅法名。在其他切⾯中使⽤必须是全限定⽅法名。
  */
  @Pointcut("execution(* com.lagou.service.impl.*.*(..))")
  public void pointcut(){}
  @Before("pointcut()")
  public void beforePrintLog(JoinPoint jp){
    Object[] args = jp.getArgs();
    System.out.println("前置通知：beforePrintLog，参数是："+ Arrays.toString(args));
  }
  @AfterReturning(value = "pointcut()",returning = "rtValue")
  public void afterReturningPrintLog(Object rtValue){
    System.out.println("后置通知：afterReturningPrintLog，返回值是："+rtValue);
  }
  @AfterThrowing(value = "pointcut()",throwing = "e")
  public void afterThrowingPrintLog(Throwable e){
    System.out.println("异常通知：afterThrowingPrintLog，异常是："+e);
  }
  @After("pointcut()")
  public void afterPrintLog(){
    System.out.println("最终通知：afterPrintLog");
  }
  /**
  * 环绕通知
  * @param pjp
  * @return
  */
  @Around("pointcut()")
  public Object aroundPrintLog(ProceedingJoinPoint pjp){
    //定义返回值
    Object rtValue = null;
    try{
      //前置通知
      System.out.println("前置通知");
      //1.获取参数
      Object[] args = pjp.getArgs();
      //2.执⾏切⼊点⽅法
      rtValue = pjp.proceed(args);
      //后置通知
      System.out.println("后置通知");
    }catch (Throwable t){
      //异常通知
      System.out.println("异常通知");
      t.printStackTrace();
    }finally {
      //最终通知
      System.out.println("最终通知");
    }
    return rtValue;
  }
}
```

#### **3.4.3** 注解模式

在使⽤注解驱动开发aop时，我们要明确的就是，是注解替换掉配置⽂件中的下⾯这⾏配置：

```xml
<!--开启spring对注解aop的⽀持-->
<aop:aspectj-autoproxy/>
```

在配置类中使⽤如下注解进⾏替换上述配置

```java
@Configuration
@ComponentScan("com.lagou")
@EnableAspectJAutoProxy //开启spring对注解AOP的⽀持
public class SpringConfiguration {
}
```



### **3.5 Spring声明式事务的⽀持**

编程式事务：在业务代码中添加事务控制代码，这样的事务控制机制就叫做编程式事务

声明式事务：通过xml或者注解配置的⽅式达到事务控制的⽬的，叫做声明式事务

#### **3.5.1** 事务的传播⾏为

![image-20211030220719260](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2021.10.30/image-20211030220719260.png)



```java
public interface PlatformTransactionManager {
	/**
	* 获取事务状态信息
	*/
	TransactionStatus getTransaction(@Nullable TransactionDefinition definition)throws TransactionException;
	/**
	* 提交事务
	*/
	void commit(TransactionStatus status) throws TransactionException;
	/**
	* 回滚事务
	*/
	void rollback(TransactionStatus status) throws TransactionException; 
}
```





## 4. 设计模式

JDK 中用到了那些设计模式?Spring 中用到了那些设计模式?这两个问题，在面试中比较常见。我在网上搜索了一下关于 Spring 中设计模式的讲解几乎都是千篇一律，而且大部分都年代久远。所以，花了几天时间自己总结了一下，由于我的个人能力有限，文中如有任何错误各位都可以指出。另外，文章篇幅有限，对于设计模式以及一些源码的解读我只是一笔带过，这篇文章的主要目的是回顾一下 Spring 中的设计模式。

Design Patterns(设计模式) 表示面向对象软件开发中最好的计算机编程实践。 Spring 框架中广泛使用了不同类型的设计模式，下面我们来看看到底有哪些设计模式?

### 4.1 控制反转(IoC)和依赖注入(DI)

**IoC(Inversion of Control,控制反转)** 是Spring 中一个非常非常重要的概念，它不是什么技术，而是一种解耦的设计思想。它的主要目的是借助于“第三方”(Spring 中的 IOC 容器) 实现具有依赖关系的对象之间的解耦(IOC容器管理对象，你只管使用即可)，从而降低代码之间的耦合度。**IOC 是一个原则，而不是一个模式，以下模式（但不限于）实现了IoC原则。**

[![ioc-patterns](https://camo.githubusercontent.com/92e6f17f500f63a340b52ef769384a924eff1bc414b16f5f2a663f38326c11e0/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f696f632d7061747465726e732e706e67)](https://camo.githubusercontent.com/92e6f17f500f63a340b52ef769384a924eff1bc414b16f5f2a663f38326c11e0/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f696f632d7061747465726e732e706e67)

**Spring IOC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** IOC 容器负责创建对象，将对象连接在一起，配置这些对象，并从创建中处理这些对象的整个生命周期，直到它们被完全销毁。

在实际项目中一个 Service 类如果有几百甚至上千个类作为它的底层，我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IOC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。关于Spring IOC 的理解，推荐看这一下知乎的一个回答：https://www.zhihu.com/question/23277575/answer/169698662 ，非常不错。

**控制反转怎么理解呢?** 举个例子："对象a 依赖了对象 b，当对象 a 需要使用 对象 b的时候必须自己去创建。但是当系统引入了 IOC 容器后， 对象a 和对象 b 之前就失去了直接的联系。这个时候，当对象 a 需要使用 对象 b的时候， 我们可以指定 IOC 容器去创建一个对象b注入到对象 a 中"。 对象 a 获得依赖对象 b 的过程,由主动行为变为了被动行为，控制权反转，这就是控制反转名字的由来。

**DI(Dependecy Inject,依赖注入)是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。**

### 4.2 工厂设计模式

Spring使用工厂模式可以通过 `BeanFactory` 或 `ApplicationContext` 创建 bean 对象。

**两者对比：**

- `BeanFactory` ：延迟注入(使用到某个 bean 的时候才会注入),相比于`ApplicationContext` 来说会占用更少的内存，程序启动速度更快。
- `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。`BeanFactory` 仅提供了最基本的依赖注入支持，` ApplicationContext` 扩展了 `BeanFactory` ,除了有`BeanFactory`的功能还有额外更多功能，所以一般开发人员使用` ApplicationContext`会更多。

ApplicationContext的三个实现类：

1. `ClassPathXmlApplication`：把上下文文件当成类路径资源。
2. `FileSystemXmlApplication`：从文件系统中的 XML 文件载入上下文定义信息。
3. `XmlWebApplicationContext`：从Web系统中的XML文件载入上下文定义信息。

Example:

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
 
public class App {
	public static void main(String[] args) {
		ApplicationContext context = new FileSystemXmlApplicationContext(
				"C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");
 
		HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
		obj.getMsg();
	}
}
```

### 4.3 单例设计模式

在我们的系统中，有一些对象其实我们只需要一个，比如说：线程池、缓存、对话框、注册表、日志对象、充当打印机、显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常、资源使用过量、或者不一致性的结果。

**使用单例模式的好处:**

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
- 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

**Spring 中 bean 的默认作用域就是 singleton(单例)的。** 除了 singleton 作用域，Spring 中 bean 还有下面几种作用域：

- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

**Spring 实现单例的方式：**

- xml : `<bean id="userService" class="top.snailclimb.UserService" scope="singleton"/>`
- 注解：`@Scope(value = "singleton")`

**Spring 通过 `ConcurrentHashMap` 实现单例注册表的特殊方式实现单例模式。Spring 实现单例的核心代码如下**

```
// 通过 ConcurrentHashMap（线程安全） 实现单例注册表
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // 检查缓存中是否存在实例  
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //...省略了很多代码
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //...省略了很多代码
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    //将对象添加到单例注册表
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

### 4.4 代理设计模式

#### 4.4.1代理模式在 AOP 中的应用

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP 就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

[![SpringAOPProcess](https://camo.githubusercontent.com/2948f9b2b5c45eb208990afcac1bf5638783dc3ebc834d3d7d2d4f7cc050981c/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f537072696e67414f5050726f636573732e6a7067)](https://camo.githubusercontent.com/2948f9b2b5c45eb208990afcac1bf5638783dc3ebc834d3d7d2d4f7cc050981c/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f537072696e67414f5050726f636573732e6a7067)

当然你也可以使用 AspectJ ,Spring AOP 已经集成了AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

#### 4.4.2 Spring AOP 和 AspectJ AOP 有什么区别?

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

### 5. 模板方法

模板方法模式是一种行为设计模式，它定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。 模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤的实现方式。

```
public abstract class Template {
    //这是我们的模板方法
    public final void TemplateMethod(){
        PrimitiveOperation1();  
        PrimitiveOperation2();
        PrimitiveOperation3();
    }

    protected void  PrimitiveOperation1(){
        //当前类实现
    }
    
    //被子类实现的方法
    protected abstract void PrimitiveOperation2();
    protected abstract void PrimitiveOperation3();

}
public class TemplateImpl extends Template {

    @Override
    public void PrimitiveOperation2() {
        //当前类实现
    }
    
    @Override
    public void PrimitiveOperation3() {
        //当前类实现
    }
}
```

Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。一般情况下，我们都是使用继承的方式来实现模板模式，但是 Spring 并没有使用这种方式，而是使用Callback 模式与模板方法模式配合，既达到了代码复用的效果，同时增加了灵活性。

### 6. 观察者模式

观察者模式是一种对象行为型模式。它表示的是一种对象与对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。Spring 事件驱动模型就是观察者模式很经典的一个应用。Spring 事件驱动模型非常有用，在很多场景都可以解耦我们的代码。比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。

#### 6.1 Spring 事件驱动模型中的三种角色

##### 6.1.1事件角色

`ApplicationEvent` (`org.springframework.context`包下)充当事件的角色,这是一个抽象类，它继承了`java.util.EventObject`并实现了 `java.io.Serializable`接口。

Spring 中默认存在以下事件，他们都是对 `ApplicationContextEvent` 的实现(继承自`ApplicationContextEvent`)：

- `ContextStartedEvent`：`ApplicationContext` 启动后触发的事件;
- `ContextStoppedEvent`：`ApplicationContext` 停止后触发的事件;
- `ContextRefreshedEvent`：`ApplicationContext` 初始化或刷新完成后触发的事件;
- `ContextClosedEvent`：`ApplicationContext` 关闭后触发的事件。

[![ApplicationEvent-Subclass](https://camo.githubusercontent.com/61adb611ba7c950b20dc6f2bc3aaddd7596497f9e8ffe1e682b822cc33793a48/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f4170706c69636174696f6e4576656e742d537562636c6173732e706e67)](https://camo.githubusercontent.com/61adb611ba7c950b20dc6f2bc3aaddd7596497f9e8ffe1e682b822cc33793a48/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f4170706c69636174696f6e4576656e742d537562636c6173732e706e67)

##### 6.1.2事件监听者角色

`ApplicationListener` 充当了事件监听者角色，它是一个接口，里面只定义了一个 `onApplicationEvent（）`方法来处理`ApplicationEvent`。`ApplicationListener`接口类源码如下，可以看出接口定义看出接口中的事件只要实现了 `ApplicationEvent`就可以了。所以，在 Spring中我们只要实现 `ApplicationListener` 接口的 `onApplicationEvent()` 方法即可完成监听事件

```
package org.springframework.context;
import java.util.EventListener;
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E var1);
}
```

##### 6.1.3 事件发布者角色

`ApplicationEventPublisher` 充当了事件的发布者，它也是一个接口。

```
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object var1);
}
```

`ApplicationEventPublisher` 接口的`publishEvent（）`这个方法在`AbstractApplicationContext`类中被实现，阅读这个方法的实现，你会发现实际上事件真正是通过`ApplicationEventMulticaster`来广播出去的。具体内容过多，就不在这里分析了，后面可能会单独写一篇文章提到。

#### 6.2 Spring 的事件流程总结

1. 定义一个事件: 实现一个继承自 `ApplicationEvent`，并且写相应的构造函数；
2. 定义一个事件监听者：实现 `ApplicationListener` 接口，重写 `onApplicationEvent()` 方法；
3. 使用事件发布者发布消息: 可以通过 `ApplicationEventPublisher `的 `publishEvent()` 方法发布消息。

Example:

```java
// 定义一个事件,继承自ApplicationEvent并且写相应的构造函数
public class DemoEvent extends ApplicationEvent{
    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source,String message){
        super(source);
        this.message = message;
    }

    public String getMessage() {
         return message;
          }

    
// 定义一个事件监听者,实现ApplicationListener接口，重写 onApplicationEvent() 方法；
@Component
public class DemoListener implements ApplicationListener<DemoEvent>{

    //使用onApplicationEvent接收消息
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMessage();
        System.out.println("接收到的信息是："+msg);
    }

}
// 发布事件，可以通过ApplicationEventPublisher  的 publishEvent() 方法发布消息。
@Component
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message){
        //发布事件
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}
```

当调用 `DemoPublisher `的 `publish()` 方法的时候，比如 `demoPublisher.publish("你好")` ，控制台就会打印出:`接收到的信息是：你好` 。

### 7. 适配器模式

适配器模式(Adapter Pattern) 将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。

#### 7.1 spring AOP中的适配器模式

我们知道 Spring AOP 的实现是基于代理模式，但是 Spring AOP 的增强或通知(Advice)使用到了适配器模式，与之相关的接口是`AdvisorAdapter `。Advice 常用的类型有：`BeforeAdvice`（目标方法调用前,前置通知）、`AfterAdvice`（目标方法调用后,后置通知）、`AfterReturningAdvice`(目标方法执行结束后，return之前)等等。每个类型Advice（通知）都有对应的拦截器:`MethodBeforeAdviceInterceptor`、`AfterReturningAdviceAdapter`、`AfterReturningAdviceInterceptor`。Spring预定义的通知要通过对应的适配器，适配成 `MethodInterceptor`接口(方法拦截器)类型的对象（如：`MethodBeforeAdviceInterceptor` 负责适配 `MethodBeforeAdvice`）。

#### 7.2 spring MVC中的适配器模式

在Spring MVC中，`DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由`HandlerAdapter` 适配器处理。`HandlerAdapter` 作为期望接口，具体的适配器实现类用于对目标类进行适配，`Controller` 作为需要适配的类。

**为什么要在 Spring MVC 中使用适配器模式？** Spring MVC 中的 `Controller` 种类众多，不同类型的 `Controller` 通过不同的方法来对请求进行处理。如果不利用适配器模式的话，`DispatcherServlet` 直接获取对应类型的 `Controller`，需要的自行来判断，像下面这段代码一样：

```
if(mappedHandler.getHandler() instanceof MultiActionController){  
   ((MultiActionController)mappedHandler.getHandler()).xxx  
}else if(mappedHandler.getHandler() instanceof XXX){  
    ...  
}else if(...){  
   ...  
}  
```

假如我们再增加一个 `Controller`类型就要在上面代码中再加入一行 判断语句，这种形式就使得程序难以维护，也违反了设计模式中的开闭原则 – 对扩展开放，对修改关闭。

### 8. 装饰者模式

装饰者模式可以动态地给对象添加一些额外的属性或行为。相比于使用继承，装饰者模式更加灵活。简单点儿说就是当我们需要修改原有的功能，但我们又不愿直接去修改原有的代码时，设计一个Decorator套在原有代码外面。其实在 JDK 中就有很多地方用到了装饰者模式，比如 `InputStream`家族，`InputStream` 类下有 `FileInputStream` (读取文件)、`BufferedInputStream` (增加缓存,使读取文件速度大大提升)等子类都在不修改`InputStream` 代码的情况下扩展了它的功能。

[![装饰者模式示意图](https://camo.githubusercontent.com/92d5fca30ccf6e54940a36521360ef71aa260a0e84875f705c966311a506f435/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f4465636f7261746f722e6a7067)](https://camo.githubusercontent.com/92d5fca30ccf6e54940a36521360ef71aa260a0e84875f705c966311a506f435/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f4465636f7261746f722e6a7067)

Spring 中配置 DataSource 的时候，DataSource 可能是不同的数据库和数据源。我们能否根据客户的需求在少修改原有类的代码下动态切换不同的数据源？这个时候就要用到装饰者模式(这一点我自己还没太理解具体原理)。Spring 中用到的包装器模式在类名上含有 `Wrapper`或者 `Decorator`。这些类基本上都是动态地给一个对象添加一些额外的职责

### 9. 总结

Spring 框架中用到了哪些设计模式？

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`