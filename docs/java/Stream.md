### 所谓Stream

简而言之：Stream就是保存在某个阶段，原始数据如果经过之前方法处理后的状态。只有执行终止操作，流才会开始处理数据，并且创建对应需要的对象与分配空间，否则一直处于一种我如果这样处理数据会怎么样的状态，就是保存接下来要执行的处理流程的方法，但实际还没有开始执行。

Stream并不保存需要处理的原始数据，执行List.srtream()方法类似于 在sql中执行一次select * from tablename，等价于获取一次视图，但它并不保存数据，数据还是保存在原始集合对象中，所以创建Stream但还没有执行时改变集合的元素将印象结果吗，Stream只是通过迭代将它们的值取出来，这些值是选择输出还是经过一些列转换变成其他类型的数据保存起来取决于接下来执行的处理流程，也就是各个方法 

```java
List.stream().filter().map().anyMatch()；  
//它可以理解为在一次迭代所有集合元素的过程中执行filter方法，而在filter方法中又执行了map方法，而在map方法中又执行了anyMatch方法
 foreach(Item item: list){
  fiter(){
    map(){
        anyMatch();
    }
}
```

流的主要作用就是进行数据处理，它减少了java8以前处理数据时的迭代次数，并提供通用的api方便使用

选择什么样的终止操作和有状态中间操作决定最后返回的类型与在开始处理数据前创建的对象类型。

举一些列子：
假设原始数据

```java
List<> list=new ArrayList<String>(Arrays.asList("shf","sb","ekst","hello"));
```

第一个例子：

```java
list.stream()
    .filter(str->str.length()>=3)
    .map(str->str.toUpCase())
    .forEach(System.out::println);
//它等价与以下方法
for(String str:list){
    if(str.length()>=3){	//filter方法的代码处理逻辑
       String s1= str.toUpCase(); //map方法的代码逻辑
        System.out.println(s1);   //foreach的代码逻辑   
    }
}
```

第二个例子

```java
int count=list.stream()
    .map(str->str.toUpCase())
    .filter(str->str.startWith("S"))
    .count();
//他等价与以下方法
int count=0;
for(String str:list){ 			//stream的代码逻辑
    String s1=str.toUpCase();	//map的代码处理逻辑
    if(s1.startWith("S")){		//filter的代码处理逻辑
        count++;				//count的代码处理逻辑
    }
}
```

第三个例子：

这个例子与前两个有些不同，他创建了额外的空间保存过滤后的数据，因为sort会影响之后数据处理，而且要进行排序，他必须等待前面过滤好数据，得到全部的数据再进行排序才有意义。这个流处理流程执行了多次迭代，在性能上与我们不使用Stream没有太大区别，但明显减少了代码的书写且逻辑结构明显。说明Stream只能减少迭代次数，并不能完全做到只迭代一次，还需要看选择的处理流程。而且由此明显可以看出利用foreach进行输出明显只是一个副作用，整个处理流程期待的结果是对源集合进行过滤并排序，然后返回新的集合，这里不做返回，而仅仅进行输出，弱化了原有的功能，所以称之为副作用。如果将foreach替换成collect,前面的代码逻辑不用变，只要将tmpList返回即可。

```java
list.stream()
    .filter(str->str.length()>=3)
    .sort()
    .forEach(System.out::println);
//他等价与以下方法
 List<String> tmpList=new ArrayList<>();
for(String str:list){
    if(str.length()>=3){	//filter方法的代码处理逻辑
       tmpList.add(str)
    }
}
Arrays.sort(tmpList);
for(String str:tmpList){
    System.out.println(str);
}
```

### 所谓无限流

就是无限迭代

```java
for(;;){
	//处理逻辑
	//......
	//终止操作,跳出循环
}
如：stream.generator(Math::random).limit(7).foreach(System.out::println);
等同于
List<Double> list=new ArrayList<>();
for(;;){
	Double num=Math.random();
	if(list.getSize()<=7){
		list.add(Double.valueOf(num));
        System.out.println(num);
	}
    else{
        break;
    
}
```

