# Java

## Java特性历史

- JDK5  泛型、枚举、自动装箱/拆箱、可变参数、注解、foreach循环、静态导入、新的线程模型（Concurrent）
- JDK7 异常处理（捕获多异常)、自动资源释放(trywithResource)）、泛型类型推断、swtich、forkjoin、G1
- JDK8 lamda、stream、dateTime、
- JDK9 模块化

  

## 类加载器

负责加载类，通常将二进制文件中的字节流转换为Class。

加载class的通用流程：

> 1. 调用`findLoadedClass(String)`方法检查这个类是否被加载过
>
> 2. 使用**父加载器**调用`loadClass(String)`方法，如果父加载器为`Null`，类加载器装载虚拟机内置的加载器
>
> 3. 调用`findClass(String)`方法装载类
>
>    如果，按照以上的步骤成功的找到对应的类，并且该方法接收的`resolve`参数的值为`true`,那么就调用`resolveClass(Class)`方法来处理类。
>    `ClassLoader`的子类最好覆盖`findClass(String)`而不是这个方法。
>    **除非被重写，这个方法默认在整个装载过程中都是同步的（线程安全的）。**

JVM类采用的是双亲委托类加载机制，目前内置的类加载器

- Bootstrap ClassLoader
- ExtClassLoader
- AppClassLoader
- 自定义ClassLoader， 一般继承URLClassLoader

JDK9以后的Classloader

- BootClassLoader
- Platform Class Loade
- AppClassLoader
- BuiltinClassloader替代URLClassLoader

springboot的类加载过程应该是AppClassLoader加载org目录下的所有类文件，再由JarLauncher创建LaunchedClassLoader（父类加载器是AppClassLoader）作为默认的类加载器去加载BOOT-INF/classes/和BOOT-INF/lib/中的类和第三方库，并运行start-class中的main方法启动spring boot应用。

对于任意一个类，都必须由加载它的**类加载器**和这个**类**本身一起共同确立其在Java虚拟机中的**唯一性**。

ContextClassloader

- 线程上下文加载器是为了解决父加载器，想要使用子加载器的场景，因为要获取子加载器对应classpath下的文件时，只有获取到子加载器。
- 通过java.lang.Thread类的setContextClassLoader()方法进行设置

插件式Classloader实现

http://xxgblog.com/2013/07/04/java-urlclassloader-plugin/

## 异常

异常类型

- 检查性异常（Checked Exception）

  非继承RuntimeException

  检查异常必须捕获或者声明再抛出

- 非检查性异常 （Unchecked Exception）

  继承RuntimeException

## 并发

线程池



## 泛型

### 类型擦除

- java伪泛型，编译器语法糖， 运行时无泛型信息，无缝使用旧代码

- class类文件的Signature属性

  > Signature 属性是可选的定长属性，位于 ClassFile， field_info 或 method_info结构的属性表中。
  >
  > 在 Java 语言中，任何类、 接口、 初始化方法或成员的泛型签名如果包含了类型变量（ Type Variables） 或参数化类型（ Parameterized Types），则 Signature 属性会为它记录泛型签名信息。

- 获取类型信息 反射

### 泛型不变、协变和逆变

- 概念

  逆变与协变用来描述类型转换（type transformation）后的继承关系，其定义：如果𝐴、𝐵表示类型，𝑓(⋅)表示类型转换，≤表示继承关系（比如，𝐴≤𝐵表示𝐴是由𝐵派生出来的子类）；

  - 𝑓(⋅)是逆变（contravariant）的，当𝐴≤𝐵时有𝑓(𝐵)≤𝑓(𝐴)成立；
  - 𝑓(⋅)是协变（covariant）的，当𝐴≤𝐵时有𝑓(𝐴)≤𝑓(𝐵)成立；
  - 𝑓(⋅)是不变（invariant）的，当𝐴≤𝐵时上述两个式子均不成立，即𝑓(𝐴)与𝑓(𝐵)相互之间没有继承关系。

- 示例

  - ArrayList<Number>这种形式的泛型是不变的，就是说ArrayList<Number> list，不能被赋值为ArrayList<Integer>，也不能被赋值为ArrayList<Object>，只能被赋值为ArrayList<Number>
  - ArrayList<? extends Number>这种形式的泛型是支持协变的，它可以被赋值为ArrayList<Number>、ArrayList<Integer>，但是不能被赋值为ArrayList<Object>
  - ArrayList<? super Number>这种形式的泛型是支持逆变的，它可以被赋值为ArrayList<Number>、ArrayList<Object>，但是不能被赋值为ArrayList<Integer>

- 最佳实践

  Producer-Extends, Consumer-Super

  PECS总结：

  - 要从泛型类取数据时，用extends；
  - 要往泛型类写数据时，用super；
  - 既要取又要写，就不用通配符（即extends与super都不用）。

```java
	/**
       * 协变， 不能写入除null的值
       * 可赋值Number及其父类的列表， 获取的对象类型为Number
       */
      List<? extends Number> numbers; 
      List<Integer> integers = new ArrayList<>(2);
      numbers = integers;
      integers.add(Integer.valueOf(0));
      integers.add(Integer.valueOf(1));
      numbers.add(null);
      //numbers.add(1); 报错
      Number n = numbers.get(0);
      Integer i = (Integer) numbers.get(0); //需要转型

  	/**
       * 逆变， 写入Nmber及其子类
       * 可赋值Number及其父类的列表， 获取的对象类型为Object
       */
      List<? super Number> numbers2 ;
      List<Number> ns = new ArrayList<>();
      numbers2 = ns;
      numbers2.add(Integer.valueOf(99));
      numbers2.add(Long.valueOf(100000L));
      Number num = (Number) numbers2.get(0);
```



### 泛型类型

- ParameterizedType

  ```java
  public interface ParameterizedType extends Type {
  　　　//获取<>中的实际类型
      Type[] getActualTypeArguments();
  　　　//获取<>前的实际类型
      Type getRawType();
  　　//如果这个类是某个类的所属,返回这个所属的类,否则返回null
      Type getOwnerType();
  }
  ```

- GenericArrayType

  泛型数组类型

  ```java
  public interface GenericArrayType extends Type {
      //获取“泛型数组”中元素的类型
      Type getGenericComponentType();
  }
  ```

  

- TypeVariable　　

  类型变量，描述类型，表示泛指任意或相关一类类型

  ```java
  public interface TypeVariable<D extends GenericDeclaration> extends Type {   
      //获得泛型的上限，若未明确声明上边界则默认为Object
      Type[] getBounds();    
      //获取声明该类型变量实体(即获得类、方法或构造器名)*
      D getGenericDeclaration();    
      //获得名称，即K、V、E之类名称*    
      String getName(); 
  }
  ```

  

- Class

```java
Result<List<Item>> aResult;

Result<T> bResult;

List<Item>[] lists;

@Test
public void testGenericType() throws NoSuchFieldException, SecurityException {
	Type t = DemoApplicationTests.class.getDeclaredField("aResult").getGenericType();
	ParameterizedType pt = (ParameterizedType)t;
    Type[] types = pt.getActualTypeArguments();
    for( int i = 0; i < types.length; i++ ){
          printType(types[i].toString(), types[i]);
   }
    
   t = DemoApplicationTests.class.getDeclaredField("bResult").getGenericType();
   pt = (ParameterizedType)t;
   types = pt.getActualTypeArguments();
   for( int i = 0; i < types.length; i++ ){
          printType(types[i].toString(), types[i]);
   }
   
  t = DemoApplicationTests.class.getDeclaredField("lists").getGenericType();
  GenericArrayType gt = (GenericArrayType)t;
  Type componentType = gt.getGenericComponentType();
  printType(componentType.toString(), componentType);
}
```


### JSON泛型反序列化

- fastJson

  ```java
  @Data
  public class Result<T> {
      private int ret;
      private String msg;
      private T data;
      
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public static class Item extends  BaseDTO {
          private String name;
          private String value;
      }
      
      @Data
      public static class BaseDTO  {
          private String id;
          private String createTime;
      }
  }
  
  /**
  * 多层嵌套泛型类Result<List<Item>>
  * 注意：ParameterizedTypeImpl有静态缓存可能OOM
  **/
  private static Type buildType(Type... types) {
      ParameterizedTypeImpl beforeType = null;
      if (types != null && types.length > 0) {
          for (int i = types.length - 1; i > 0; i--) {
              beforeType = new ParameterizedTypeImpl(new Type[]{beforeType == null ? types[i] : beforeType}, null, types[i - 1]);
          }
      }
      return beforeType;
  }
  
  JSON.parseObject(json, new TypeReference<Result<Item>>(){})
  //多层嵌套泛型类
  JSON.parseObject(json, buildType(Result.class, List.class, BaseDTO.class));
  
  
  ```

##  Lamda

## 函数式编程

函数称为第一类的类型， 可以作为函数的参数，赋值给变量及返回值。

```java
@Test
public void testLamda() {
	final Collection< Task > tasks = Lists.newArrayList(
				    new Task( Status.OPEN, 5 ),
				    new Task( Status.OPEN, 13 ),
				    new Task( Status.CLOSED, 8 )

		);
		
	Integer openSum = tasks.stream()
			.filter(Task::opening)
			.mapToInt(Task::getPoints)
			.sum();
		
	System.out.println("sum:" + openSum);

	Map<Status, Integer> groupSum = tasks.stream()
				.collect(Collectors.groupingBy(Task::getStatus, Collectors.summingInt(Task::getPoints)));
		
	System.out.println("groupSum:" + groupSum);
		
	Map<Status, List<Task>> partition = tasks.stream()
				.collect( Collectors.groupingBy(Task::getStatus, Collectors.toList()));
	System.out.println("partition:" + partition);

	}
```

### Stream

> Java 8 的 Stream 主要关注在流的过滤，映射，合并，而 Reactive Stream 更进一层，侧重的是流的产生与消费，即流在生产与消费者之间的协调

### Reactive Stream

```java
// 发布者(生产者)
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
// 订阅者(消费者)
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
// 用于发布者与订阅者之间的通信(实现背压：订阅者能够告诉生产者需要多少数据)
public interface Subscription {
    public void request(long n);
    public void cancel();
}
// 用于处理发布者 发布消息后，对消息进行处理，再交由消费者消费
public interface Processor<T,R> extends Subscriber<T>, Publisher<R> {
}
```

### Reactor

- Mono

  Mono<T>是一种专门的发布器（Publisher<T>），它通过onNext信号最多发出一个项目，然后以onComplete信号终止（成功的Mono，有值或无值），或者只发出一个onError信号（失败的Mono）

- Flux

  Flux<T>是一个标准发布器(Publisher<T>)，它表示由0到N个发出的项组成的异步序列，可以选择由完成信号或错误终止。在reactivestreams规范中，这三种类型的信号转换为对下游用户的onNext、onComplete和onError方法的调用。

- publishOn

  它接收来自上游的信号，并在下游重放这些信号，同时对来自关联调度器的worker执行回调。因此，它会影响后续操作符的执行（直到另一个publishOn链接进来）

  ```java
  Scheduler s = Schedulers.newParallel("parallel-scheduler", 4); 
  final Flux<String> flux = Flux    
  	.range(1, 2)    
      .map(i -> 10 + i)  // thread1 执行
      .publishOn(s) // threadx in scheduler
      .map(i -> "value " + i);   // threadx 执行
  
  new Thread(() -> flux.subscribe(System.out::println));  // subscription 发生在 thread1, print在threadx中执行
  ```

  

- subscribeOn

  它应用于subscription处理时（反向链被构造）。因此，无论您将subscribeOn放置在链中的何处，它都会影响源发射的上下文。但是，这并不影响对publishOn -的后续调用的行为；它们仍然会切换后面部分链的执行上下文。

  > 订阅链中有多个subscribeOn，只有最开始的subscribeOn起作用。

  ```java
  Scheduler s = Schedulers.newParallel("parallel-scheduler", 4);  
  final Flux<String> flux = Flux    
  	.range(1, 2)    
      .map(i -> 10 + i)  //threadx 执行  
      .subscribeOn(s)   //从订阅时间开始切换整个序列 , threadx 
      .map(i -> "value " + i);   //threadx执行
  new Thread(() -> flux.subscribe(System.out::println)); //Thread中订阅，但subscribeOn切换到调度线程 threadx， print在threadx执行
  ```

   