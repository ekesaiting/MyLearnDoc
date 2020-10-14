### 包装类中的基础类型的值不可变

不能使用包装类对象作为函数参数而又期待在函数内改变他的值，属性value被final修饰不能改变。

### 子类重写的方法的访问修饰符必须大于等于父类

返回值必须等于或是父类方法返回值的子类，子类抛出的异常必须小于父类，即父类不抛出异常，子类也无法抛出异常，典型如Runable接口

### 所有集合类型只能存储对象

基本数据类型会自动装箱与拆箱。只要声明是包装类型即可，添加时可以是基本类型也可以是包装类型

### object是所有对象的父类

即使是Object[]数组对象，Class<>，，int[],枚举类等等。注意接口虽然没有父类，但它又有一套隐含的与Object相同方法签名的方法，即不能在接口中定义`int boolean equals(Object obj);`等方法，因为该接口已经隐含了一个相同方法签名的方法，而不同的返回值不能作为方法重载的依据，可以将`int`修改为`boolean`覆盖这个隐含的方法

### Class<>对象相当于一个类的描述对象

**拥有了Class对象，就能对该类与他的实例做任何事**

也可以理解为类的元数据对象。他记录类的类加载器，访问修饰符 ，包名，全限定类名，继承的类，实现的接口，被标注的注解，抛出的异常类型，拥有的成员变量，构造方法与普通方法。注意：基础数据类型也有相对应的描述对象，如：int.class,他等同于Integer.TYPE 返回的都是同一个Class对象，但Integer.class返回的是Class<Integer>类型的对象，与以上两种不等。八种基本数据类型与void将由jvm创建对应的Class对象，主要用于反射。

### Class对象也可以加载资源

典型的两个方法：getResource()和getResourceAsStream(),并且他们不会抛出异常或IO错误，找不到资源只是返回值为空。

### HashSet无序

所谓的HashSet无序指的是插入的顺序与遍历的顺序得到的结果不一致，对于同一set,两次遍历的结果是一致的，都是从底层的数组下标为零开始。（只要插入的对象hashcode值进过hash()混淆后得到的数组下标值恰好还是原来顺序的话还是可以得到与插入顺序一致的结果的，hashset只是不保证有序）

### 所谓符号引用

就是在一个类中使用了其他类，在这个类运行时，他引用的类必须加载到jvm中，但是这个类文件并不含有这个引用的类信息，他只有一个引用（如System.out.println(),我们自己的类文件中肯定没有System类的信息,他只有一个符号 java.lang.System），告诉jvm我需要用它，jvm会帮我们把它加载进来，并将这个符号指向内存中方法区中这个存放这个类信息的起始位置，如在此后的代码中还出现这个符号，jvm可以通过直接引用直接获取这个类的信息。

### java匿名内部类与lambda表达式的区别

* 匿名内部类还是一个类，在编译后会生成一个class文件，由jvm根据一定规则给一个名字，
* 匿名内部类内部的方法使用的局部变量必须由final修饰，没有的话编译器会默认添加，（成员变量与全局变量不必），不然随着方法的结束，栈帧消失，而这个匿名内部类对象的引用传递给了其他对象，这个匿名内部类对象不会被回收，但她引用的局部变量已近被回收了，所以引用的局部变量必须为final，在引用时拷贝一份传递给匿名内部类中的方法
* lamdba表达式内部使用的局部变量无需使用final修饰，编译器也不会添加final
* 匿名内部类与lamdba的方法体内都不能修改局部变量，无论是否有final修饰，改变的话不能通过编译。

```java
new Thread(new Runable(){
    public void run(){
        System.out.println("Lambda Thread run()")
    }
}).start();
//以上代码会生成一个内部类。并且调用他的默认构造函数生成一个新对象作为参数传递给Thread的构造函数
```

lambda表达式不会生成一个类，而是在当前类中生成一个方法

```java
public class MainLambda {
	public static void main(String[] args) {
		new Thread(
				() -> System.out.println("Lambda Thread run()")
			).start();;
	}
}
//反编译之后我们发现Lambda表达式被封装成了主类的一个私有方法，并通过invokedynamic指令进行调用。
//因为没有类，也就没有对象，在lambda表达式中的this指针相当于MainClass中的一个实例方法中的this,代表当前对象，与lambda表达式外的this完全一样
```

### 不要使用Stack

VSHE：Vector,Stack,HashTable,Enumration线程安全

java已经放弃了Stack,因为他继承了Vector,要使用`队列`或者`栈`这种数据结构请使用==ArrayDeque==

当做队列使用：入队：offerLast，出队：pollFisrt

当做栈使用：入栈：offerLast，出栈：pollLast



### idea项目启动

他在当前项目的根目录 使用java 命令启动，并添加一系列参数，如maven 依赖，在未打包的情况下，他只存在与本地maven中，idea将依赖的jar包路径， 通过启动参数的方式加入类路径以供应用类加载器可以扫描到并且加载到jvm中，并且java源码中任何位置的相对路径都是以项目目录开始。以`/`开头表示操作系统根目录，windows系统没有此目录，linux才有，

注意：maven项目的resource目录下的文件需要通过类加载器的getResourceAsStream获取，并且此时的相对路径为resource目录

### java IO

除FileInputStream对象的构造函数可以直接接受字符串路径或File对象外，其他包装流或转换流对象都要接受一个流对象作为构造函数参数，

PrintWriter的意思不是向屏幕输出，是以文本格式输出字符串和数字，因为Writer为抽象类，所以最常见的子类就是Printwriter，它可以任意指定输出位置，必FileWriter更全能。对应字节流PrintStream

### try finally return 说明

finally中的return 会覆盖catch中的return,效果等同于覆盖try块中的return,规则如下：

1. 如果finally中有return 语句，则将try中return 覆盖，返回finally中的return值
2. 如果finally中没有return 语句，也没有修改return的值或对象，则执行完finally继续执行try块中的return
3. 如果finally中没有return语句，但修改了返回值，
   * 如果是基本数据类型或者String,则不变，继续返回try块中的值（返回的是复制的槽位里的值，String对象因为不可变，相当于创建一个新对象并赋值给原有槽位，但最终返回的String对象时一开始复制的）
   * 如果是引用数据类型，try中的return语句返回的就是在finally中改变后的该属性的值。（因为复制局部变量表中变量引用地址后两个槽位的值是一样的，在finallly中修改对象的字段值不会修改对象本身的地址，只有将变量赋值给一个新对象，导致两个槽位的值不同，因为返回的是复制的槽位中的值，finally中的赋值将失效）

### List<int[]>

注意int[] 等所有数组类型等于引用类型，都继承自object,

### 日志门面与日志框架

  参考：https://sites.google.com/site/yeyanbo/good-idea/good-idea-java/log4j

**日志门面**：提供统一的Api操作多个日志框架，如 slf4j,commons logging，在slf4j中，都是通过 Logger log=LoggerFactory.getLogger(Object.class)，获得日志输出对象，通过如log.log()等打印日志。

**日志框架**：提供具体的输出方式，每个日志框架有不同的Api，如：log4j,logback,simple-slf4j等等

可以直接操作日志框架，典型的使用方式，导入log4j的jar包，配置log4j.properties或log4j.xml文件即可，

但如果应用使用了多个日志框架（导入了多个jar包，但他们内部使用了不同的日志框架），不同的api将导致操作的不便，所以一般较大的应用都使用日志门面，

Log4j由三个重要的组件构成：Logger、Appender和Layout，Log4j 允许开发人员定义多个Logger，每个Logger拥有自己的名字，Logger之间通过名字来表明隶属关系。有一个Logger称为Root，它永远存在，且不能通过名字检索或引用，可以通过Logger.getRootLogger()方法获得，其它Logger通过 Logger.getLogger(String name)方法。

Appender则是用来指明将所有的log信息存放到什么地方，Log4j中支持多种Appender，一个Logger可以拥有多个Appender，也就是你既可以将Log信息输出到屏幕，同时存储到一个文件中。

Layout的作用是控制Log信息的输出方式，也就是格式化输出的信息。

### System/Runtime

System虽然不是抽象类，但他的构造函数私有化，所以不能创建实例对象，使用的都是静态方法

Runtime实例代表jvm进程，一个jvm只有一个Runtime对象，采用饿汉式单例，在类加载时创建

### ==

==比较的是栈中局部变量表中的值，基本类型就是值，引用类型就是地址

### Integer.hashcode()

数值类型的包装类的hashcode值就是他们存有的基本数据类型的值,往hashset中存储两个相同的基本数据类型不被允许，虽然他们的包装类对象不同，但他们的hashcode值相同。

### 删除集合元素

遍历集合时删除集合元素不能使用foreach，虽然他底层仍然是使用iterator，但删除时用的是list.remove（）方法，这将导致并发修改异常。正确的用法是手动获取迭代器，通过while(iterator.hasNext（）)进行迭代遍历，并通过iterator.remove()进行删除，这不会导致异常抛出

### ++i与i++

当他们未出现在“=”右边，如（i++;/++i;）字节码指令没有任何区别，都是在局部变量表上自增1，如果他们参与运算，即出现在“=”右边，那么i必须进入操作数栈，如果++i,表示先在局部变量表上自增1，将自增后的结果值提取到操作数栈，即参与当前运算的是自增后的值，如果是i++,表示先提取i的值到操作数栈，然后局部变量表自增1，即参与当前运算的是原始值。