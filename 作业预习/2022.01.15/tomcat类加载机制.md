**一、tomcat类加载器关系**

​	![q0jnymcvj6](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2022.01.15/q0jnymcvj6-2222833.png)

​    tomcat类加载器设计结构如上图所示，上面三个Bootstrap加载器、Ext加载器、Application加载器是JVM加载器，下半部分的加载器才是tomcat自定义的加载器。

​    学习tomcat类加载器之前先回顾下之前写过的jvm类加载器的知识点。

**·  类加载机制**

​    类加载机制的本质是将经过虚拟机编译后的.class文件转化为二进制，并将二进制加载成Class对象的加载过程。

​    因此判断类是否完全相同，需要以下两个条件即：类的全路径名是否相同、加载类的类加载器是否为同一个，这两个条件决定类是否完全相同。

**·  双亲委派机制**

​    双亲委派机制的存在，保证类是安全的，他的存在保证了类的加载顺序会优先委派父类加载器进行加载，如果父类加载器没有加载才会有子类进行加载。比如在JVM类加载器的继承关系（从子到父）为Application ClassLoader—>Extension ClassLoader—>Bootstrap ClassLoader。因此当自己在代码内部中编写恶意java代码时，比如HashMap文件，如果路径名相同，则会委派给父类Bootstrap ClassLoader进行加载，发现在Bootstrap加载的lib目录下存在HashMap文件则加载应该由Bootstrap加载的HashMap文件。从而保证自己编写的恶意java代码不会替换应由Bootstrap加载的源文件。

**·  打破双亲委派机制**

​    由于虚拟机中的加载规则是按需加载的，即需要用到什么类的时候才会去加载那个类。并且在加载该类时用的是什么加载器，那么加载该类引用的类也需要用到对应的加载器，在java中的SPI机制，加载jdbc时由于Driver类不在rt.jar中因此不能被Bootstrap加载器进行加载，因此使用了线程上下文类加载器委派子类进行加载。所以打破了双亲委派机制，并且在tomcat类加载器中也存在打破双亲委派机制的情况。

**二、tomcat类加载器**

**·  Common ClassLoader**

  Common ClassLoader是tomcat最基本的类加载器，被此加载器加载的类即可以被tomcat所访问，也可以被应用war包中的程序所访问。

**·  Catalina ClassLoader**

   Catalina ClassLoader是tomcat私有的类加载器，被此加载器加载的类，只能被tomcat所访问。

**·  Shared ClassLoader**

​    Shared ClassLoader是各个war包共享的类加载器，被此加载器加载的类，只能被应用war包中的程序所访问。

​    当tomcat中存在多个war包并同时使用了相同版本的jar包时，为了减少资源的浪费，可以使用该加载器，抽出这些相同版本的jar包，使用Shared ClassLoader加载一次被共享的jar即可，来代替每个war包中都需要加载的过程。

**·  WebApp ClassLoader**

   WebApp ClassLoader是多个war包的类加载器，即tomcat中的一个war包由一个WebApp ClassLoader加载。

​    Shared ClassLoader和WebApp ClassLoader的存在可以让共享的jar包由Shared ClassLoader加载，其余不一样或需要独立部署的由WebApp ClassLoader进行加载，达成共享与隔离的统一。

**·  JSP ClassLoader**

​    JSP ClassLoader是一个jsp文件的类加载器，即tomcat中存在一个jsp就会new出一个JSP ClassLoader进行jsp文件的加载。

​    JSP ClassLoader是一个特殊的类加载器，他的特点也保证了jsp文件可以进行热部署，因为当对jsp文件进行修改，重新加载该jsp文件时，又会new出一个JSP ClassLoader来加载jsp文件，替换修改前的jsp文件（此时需要注意的是jsp文件的类的唯一性也变了，类的唯一性由类的全路径名+类加载器决定，此时的类加载器是重新new出来的所以和修改前的加载器是完全不一样的）。而Controller、service等文件的修改前和修改后是由相同的WebApp ClassLoader加载的，因此不能在这种情况下和jsp一样实现实现修改后的热部署。

**·  源码中的加载器关系**

```java
private void initClassLoaders() {
    try {
        //common加载器，父类为null
        commonLoader = createClassLoader("common", null);
        if (commonLoader == null) {
            commonLoader = this.getClass().getClassLoader();
        }
        //catalina加载器，父类为commonLoader
        catalinaLoader = createClassLoader("server", commonLoader);
        //shared加载器，父类为commonLoader
        sharedLoader = createClassLoader("shared", commonLoader);
    } catch (Throwable t) {
        handleThrowable(t);
        log.error("Class loader creation threw exception", t);
        System.exit(1);
    }
}
```

​    但是创建common加载器时，传入的是null，如何判断common加载器的父类是ApplicationClassLoader呢。

![g0bmo74uih](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2022.01.15/g0bmo74uih-2222935.png)

​    跟到底层会发现获取系统类加载器为父类加载器，即AppClassLoader。其余源码的分析在类加载详解中有进行过分析，这里不再做详细阐述。

**三、tomcat类加载流程**

![o7tcz7b21q](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2022.01.15/o7tcz7b21q-2222960.png)

**·  Bootstrap**

​    主要加载JVM启动所需要的类。

**·  System**

​    主要加载tomcat启动的类，比如bootstrap.jar。位于bin/bootstrap.jar。

![uq270qfvrp](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2022.01.15/uq270qfvrp-2223031.png)

**·  Common**

​    主要加载tomcat中常用的类，位于lib中。

![uqd4z31nfa](/Users/zhang/Documents/lagou/lagou_private_education/作业预习/2022.01.15/uqd4z31nfa-2222995.png)

**·  WebApp**

​    主要加载应用的类文件，即位于WEB-INF/lib下的jar文件和WEB-INF/classes下的class文件。

**四、tomcat类加载源码分析**

```java
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
}

//重写了loadClass方法
public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {

    synchronized (JreCompat.isGraalAvailable() ? this : getClassLoadingLock(name)) {
        if (log.isDebugEnabled())
            log.debug("loadClass(" + name + ", " + resolve + ")");
        Class<?> clazz = null;
        //校验程序是否启动，如果已启动抛出异常
        checkStateForClassLoading(name);
        //校验当前对象缓存是否加载过该类
        clazz = findLoadedClass0(name);
        //如果已经加载过该类 则直接返回
        if (clazz != null) {
            if (log.isDebugEnabled())
                log.debug("  Returning class from cache");
            if (resolve)
                resolveClass(clazz);
            return clazz;
        }

        //校验是否加载过该类，会调用native方法
        clazz = JreCompat.isGraalAvailable() ? null : findLoadedClass(name);
        if (clazz != null) {
            if (log.isDebugEnabled())
                log.debug("  Returning class from cache");
            if (resolve)
                resolveClass(clazz);
            return clazz;
        }


        String resourceName = binaryNameToPath(name, false);
        ClassLoader javaseLoader = getJavaseClassLoader();
        boolean tryLoadingFromJavaseLoader;
        try {

            //使用javaseLoader加载
            // 即-使用ExtClassLoader/BootstrapClassLoader判断是否需要加载这个calss
            // 防止先被webAppClassLoader加载了脏的类
            URL url;
            if (securityManager != null) {
                PrivilegedAction<URL> dp = new PrivilegedJavaseGetResource(resourceName);
                url = AccessController.doPrivileged(dp);
            } else {
                url = javaseLoader.getResource(resourceName);
            }
            tryLoadingFromJavaseLoader = (url != null);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            tryLoadingFromJavaseLoader = true;
        }

        if (tryLoadingFromJavaseLoader) {
            try {
                clazz = javaseLoader.loadClass(name);
                if (clazz != null) {
                    if (resolve)
                        resolveClass(clazz);
                    return clazz;
                }
            } catch (ClassNotFoundException e) {
            }
        }

        //安全策略校验
        if (securityManager != null) {
            int i = name.lastIndexOf('.');
            if (i >= 0) {
                try {
                    securityManager.checkPackageAccess(name.substring(0, i));
                } catch (SecurityException se) {
                    String error = sm.getString("webappClassLoader.restrictedPackage", name);
                    log.info(error, se);
                    throw new ClassNotFoundException(error, se);
                }
            }
        }

        //判断是否需要被WebAppClassLoader进行加载，是否设置了delegate
        boolean delegateLoad = delegate || filter(name, true);
        //委托给父类 CommonClassLoader加载
        if (delegateLoad) {
            if (log.isDebugEnabled())
                log.debug("  Delegating to parent classloader1 " + parent);
            try {
                clazz = Class.forName(name, false, parent);
                if (clazz != null) {
                    if (log.isDebugEnabled())
                        log.debug("  Loading class from parent");
                    if (resolve)
                        //加载成功则返回
                        resolveClass(clazz);
                    return clazz;
                }
            } catch (ClassNotFoundException e) {
            }
        }

        //使用WebAppClassLoader进行加载
        if (log.isDebugEnabled())
            log.debug("  Searching local repositories");
        try {
            clazz = findClass(name);
            if (clazz != null) {
                if (log.isDebugEnabled())
                    log.debug("  Loading class from local repository");
                if (resolve)
                    resolveClass(clazz);
                return clazz;
            }
        } catch (ClassNotFoundException e) {
        }

        //WebAppClassLoader加载失败，则委托WebAppClassLoader的父类进行加载
        if (!delegateLoad) {
            if (log.isDebugEnabled())
                log.debug("  Delegating to parent classloader at end: " + parent);
            try {
                clazz = Class.forName(name, false, parent);
                if (clazz != null) {
                    if (log.isDebugEnabled())
                        log.debug("  Loading class from parent");
                    if (resolve)
                        resolveClass(clazz);
                    return clazz;
                }
            } catch (ClassNotFoundException e) {
                // Ignore
            }
        }
    }

    //加载不到 抛出异常
    throw new ClassNotFoundException(name);
}
```

​    上述源码分析中，tomcat的类加载步骤为（delegate设置为true的情况）：

（1）判断当前被加载的类对象是否存在缓存中，如果存在则直接返回该类。

（2）判断是否加载过该类，如果加载过则直接返回该类。

（3）委派jvm中的父类ExtClassLoader、BootStrapClassLoader是否加载过该类。

（4）委派tomcat中的父类CommonClassLoader是否加载过该类。

（5）当前WebAppClassLoader自己加载该类。

（6）如果类仍然没有加载成功，则抛出异常。

**五、tomcat打破双亲委派加载机制**

​    不论加载的delegate配置设置为true或者false，tomcat加载类的顺序都为先查询当前WebAppClassLoader类加载器是否加载过该类，而不是预先委托父类进行加载。

​    第二点是当delegate设置为false时，上述的第四步和第五步可能也会打破双亲委派，优先使用WebAppClassLoader加载器进行加载，加载不到才委托父类进行加载。