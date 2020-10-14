## ThradGroup

> java ThreadGroup介绍https://juejin.im/post/6844903811899719694

子线程有异常产生而未捕获将导致子线程的退出，对象其他线程没有影响，主线程产生的异常未被捕获将导致整个jvm进程的结束(一般主线程是最后一个守护线程)，其他所有子线程对应也将结束，因此可以在创建线程时给线程指定一个未被捕获异常的处理器。

线程组是一个线程与线程组的集合，一个线程组可以包含多个线程与多个线程组（对应两个list），除了初始线程组外（System ThreadGroup），每一个线程组都有一个父线程组。

System线程组有一个子线程组 main ThreadGroup,mian ThreadGroup包含主线程（main线程），一旦一个线程归属到一个线程组之中后，就不能再更换其所在的线程组(有get,没有set方法)。

如果创建线程时没有指定该线程的线程组，他将加入创建该线程的线程的线程组【如：main线程创建的新线程在不指定线程组时的默认线程组就是main ThreadGroup】

ThreadGroup 实现了Thread.UncaughtExceptionHandler接口，因此每一个线程组都含有uncaughtException()方法的实现，它将是处理未捕获异常的方法，由于每一个线程都一定属于一个线程组，所以线程组最重要的作用就是提供一个兜底的处理未捕获异常的方法，以便正确的结束出现未捕获异常的线程

## thread.interrupt()

> Java之Interrupt解析 https://juejin.im/post/6844904094474174478

该方法将thread对象的中断标志位置为true（该标志位不是java中Thread对象的一个属性，属于c++对象级别）

* 如果线程正好正在执行`Object.wait(), Thread.join(), or Thread.sleep()`和其他重载了这些这方法时，线程会被立马中断。然后这些方法会清理`interrupted`标记位，即重新设置为false。然后抛出`InterruptedException`
* 如果线程时阻塞在网络I/O上（如`Selector.select()`）.NIO的chanel（或者socket)将会被关闭。需要注意的是，这仅仅适用JDK实现方式，而不是其他框架。

其他中断的行为都是基于这些机制，Future类，Executor，Lock和Condition的所有有关中断的行为都基于上面的机制。

synchronized在获锁的过程中是不能被中断的，意思是说如果产生了死锁（线程尝试获取锁而导致的阻塞），则不可能被中断,与synchronized功能相似的reentrantLock.lock()方法也是一样，它也不可中断的，即如果发生死锁，那么reentrantLock.lock()方法无法终止

当线程处于非阻塞状态时，除了改变标志位，对该线程没有任何影响，但可以将thread.interrupt当做一个变量来使用来终止非阻塞的线程，即使用while()判断当前线程的中断状态。

**interrupted()**方法返回当前线程的中断状态，并将中断状态置为false

线程未启动（未调用start()方法）时调用interrupt()不会改变中断状态，当线程启动时还是false

## **SecurityManager**

SecurityManager在Java中被用来检查应用程序是否能访问一些有限的资源，例如文件、套接字(socket)等等。它可以用在那些具有高安全性要求的应用程序中。通过打开这个功能， 我们的系统资源可以只允许进行安全的操作。

当Java虚拟机启动时，它首先通过检查系统属性java.security.manager来确定SecurityManager是否打开了。如果打开了，那么SecurityManager实例将被创建，它可以被用来检查不同的权限。**默认情况下，SecurityManager是关闭的**，但是这里有一些方法可以打开

* 指定 -Djava.security.manager

* ```java
  #程序中手动创建
  SecurityManager sm=new SecurityManager();
  System.setSecurityManager(sm);
  ```

## ThreadLocal

Threadlocal本身不保存设置的对象，ThreadLocal封装了ThreadLocalMap（ThreadLcoal中的一个静态内部类）,Thread有一个static的ThreadLocalMap类型的全局属性ThreadLocalMaps，他本质是一个HashMap，每次调用ThreadLocal的set/get 方法时，方法内部先获取当前线程对象，再拿到当前线程内部的threadLocalMaps 对象，将当前的ThreadLocal对象为key，实际需要保存的对象为value保存在这个线程内部的map中。如果需要保存多个对象，可以定义多个ThreadLcoal对象，每一个ThreadLocal对象都将以key的形式保存在当前线程中（ThreadLocal对象必须在每一个线程内部创建才能唯一绑定当前线程），通过ThreadLcoal的get()方法可以在当前线程的ThreadLocalmaps中获取这个ThreadLocal唯一绑定的值。