> java默认的hashcode方法到底得到的是什么？https://blog.gavinzh.com/2018/08/23/what-is-hashcode-of-java/
>
> 深入探究 JVM | klass-oop对象模型研究：https://www.sczyh30.com/posts/Java/jvm-klass-oop/
>
> Java 对象模型 OOP-Klass https://juejin.im/post/5d8425fbf265da03ee6a914c
>
> Java 多线程（二）－Monitor https://www.jianshu.com/p/6fe4bc3374a2

## hashcode方法

结论：

* OpenJDK默认的hashCode方法实现和==对象内存地址无关==。jdk 6，7版本中的hashcode是随机生成，8是基于线程的状态数字，hashcode()方法的注释描述有误。

* 32位虚拟机中对象头中25位比特存储的hashcode值（64位虚拟机hashcode值31位）就是java对象hashcode的返回值，一旦对象创建，hashcode值将不再改变，所以才保证在垃圾收集器的影响下对象的移动导致地址的改变不影响hashcode值与不大量产生重复值。

* 31位的hashcode值只有21亿种可能，但理论上只要内存够大存在无数的对象，而每一个对象都要有一个hashcode值，无限映射到有限必然导致hashcode值的重复。
* 所谓hashcode本地方法，就是java第一次调用hashcode()方法时，jvm这个c++程序会判断对象头中的hashcode是否有值，有则直接返回这个值，没有就调用相关方法生成一个填充到对象头中并返回给java程序

## klass内存模型

* HotSpot JVM 并没有根据 Java 实例对象直接通过虚拟机映射到新建的 C++ 对象，而是设计了一个 oop-klass model。

* oop是一个c++对象，但是不包含任何虚函数，对应我们创建的java对象，

* klass也是一个c++对象,包含虚函数，对应我们加载的每一个类

* 由此我们来总结一下，当我们 new 一个对象的时候，JVM 首先会判断这个类是否加载没有的话会进行加载并创建一个 instanceKlass 对象来描述 Java 类，用它可以获取到类对应的元数据信息如运行时常量池等信息，然后到初始化的时候会创建一个 instanceOopDesc 来表示这个 Java 对象实例，然后会进行 oop 的填充，包括填充对象头，使元数据指针指向 instanceKlass 类，然后填充实例数据。 **创建完成后，instanceKlass 保存在方法区中，instanceOopDesc 保存在堆中，对象的引用则保存在栈中。**

## Monitor

“监视器”是 monitor 这一词的直译，放在这里不容易理解是什么意思。
“管程”是专有名词，特指某些编程语言内建的一种互斥同步机制（相关知识在市面上的操作系统教材中都有描述）。
所以 Java 中的 monitor 应译为管程，更加准确。

Monitor可以理解为一个同步工具或一种同步机制，通常被描述为一个对象。每一个Java对象就有一把看不见的锁，称为内部锁或者Monitor锁。

Monitor是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联，同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。

现在话题回到synchronized，synchronized通过Monitor来实现线程同步，Monitor是依赖于底层的操作系统的Mutex Lock（互斥锁）来实现的线程同步。

如同我们在自旋锁中提到的“阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长”。这种方式就是synchronized最初实现同步的方式，这就是JDK 6之前synchronized效率低的原因。这种依赖于操作系统Mutex Lock所实现的锁我们称之为“重量级锁”，JDK 6中为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。



