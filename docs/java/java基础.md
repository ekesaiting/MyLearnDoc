## java的基本程序设计结构

* break:label可以跳到指定的带标签的语句块末尾，从而实现类似goto的功能

  ```
  label:
  {
  	{
  		int i=1;
  		break:label
  	}
  }//程序将跳转到这，执行下一行代码
  int b=2;
  ```

  

* 在java7中switch结构才支持字符串，如果是枚举可以不用写枚举类型，直接使用对象名。
* BigDecimal与BigInteger,理解为高精度小数与整数对象，接受Double与Long作为构造函数参数（意思就是使用Double和Long还是不能表示数据时再使用）,高精度表现在两方面：
  1. 可以表示任意长度的小数或整数
  2. 进行精确的数学运算（主要用于代替double）

## 对象和类

#### 对象的三个特征：

* 标识：在java中通过对象在内存的起始地址唯一标识
* 状态：对象中保存的实例字段
* 行为：改变对象状态的措施，java中通过类（不是对象）的方法改变

### 对象与类的关系

* 简单理解对象和类：对象和类都是内存中的一段连续的数据，对象有对象头和实际存储对象的对象体，对象头中有对象的一些元数据，如：对象年龄标识，对象锁标识，对象所属的类的内存位置。对象中只有实例字段，如构造器，方法，静态字段方法等都没有，全部存储在方法区的类数据中。
* 要访问这些方法（静态和实例）和字段（静态），通过对象头中的保存的地址可以找到方法区中类的位置，每一个类（不是每一个对象）都有一张方法表，方法表中存储了当前方法所在的内存位置起始地址，当通过对象.方法执行这个方法时，将在类方法表中找到这个方法（通过索引，也就是下标）,一个类继承另一个类时（不是对象之间的继承）将复制一份方法表，所有方法的索引位置不变，当修改子类继承的方法（重写）时，将新建一段内存数据保存这个方法，并将这段内存数据的起始地址覆盖方法表中原有方法的索引位置的值，新增方法的将添加到方法表中的下一个索引位置
* 所谓多态就是访问引用类型的类所在的虚方法表，找到该方法的索引值。在通过方法中的this指针找到对象（所有实例方法都必须由对象调用且都将传入一个代表该对象内存起始位置的指针）通过这个对象找到这个对象所属的类的方法表，在同样的索引位置找到这个方法，进而执行方法代码，
* 因此如果引用类型中没有定义相应方法，将无法在方法表中找到这个方法的索引，程序运行时将报错。强转就是改变变量的引用类型。如果随意强转，当强转后变量的引用类型中有对应方法，而实际对象所属的类没有这个方法时，程序将报错。

**java 属性多态。 访问变量看声明，访问方法看实际对象类型**

### 直接引用和符号引用

> https://blog.csdn.net/u010386612/article/details/80105951

所有方法的调用都是jvm的一条指令如：demo.run()，实际就是类似`invokevirtual #2  // Method run:()V`的一条指令，在class文件中以这种形式存在`[B6] [00 02]`其中0xB6是[invokevirtual指令](https://link.zhihu.com/?target=https%3A//docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html%23jvms-6.5.invokevirtual)的操作码（opcode），后面的0x0002是该指令的操作数（operand），用于指定要调用的目标方法。这个参数是Class文件里的常量池的下标。那么去找下标为2的常量池项

```
#2 = Methodref          #3.#17         //  X.run:()V
```

这在Class文件中的实际编码为（以十六进制表示，Class文件里使用高位在前字节序（big-endian））：

```
[0A] [00 03] [00 11]
```

其中0x0A是CONSTANT_Methodref_info的tag，后面的0x0003和0x0011是该常量池项的两个部分：class_index和name_and_type_index。这两部分分别都是常量池下标，引用着另外两个常量池项。
顺着这条线索把能传递引用到的常量池项都找出来

**由此可以看出，Class文件中的invokevirtual指令的操作数经过几层间接之后，最后都是由字符串来表示的。这就是Class文件里的“符号引用”的实态：带有类型（tag） / 结构（符号间引用层次）的字符串。**

原本常量池项#2在类加载后的运行时常量池里的内容跟Class文件里的一致，是：

```
[00 03] [00 11]
```

以后再查询到常量池项#2时，里面就不再是一个符号引用，而是一个能直接找到Java方法元数据的methodblock* 了。这里的methodblock* 就是一个“直接引用”。

解析好常量池项之后回到invokevirtual指令的解析。
回顾一下，在解析前那条指令的内容是：

[B6] [00 02]

而在解析后，这块代码被改写为：

[D6] [06] [01]

opcode部分从invokevirtual改写为invokevirtual_quick，以表示该指令已经解析完毕。



