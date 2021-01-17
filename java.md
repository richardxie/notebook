# Java

## Java特性历史

- JDK5  泛型、枚举、自动装箱/拆箱、可变参数、注解、foreach循环、静态导入、新的线程模型（Concurrent）
- JDK7 异常处理（捕获多异常)、自动资源释放(trywithResource)）、泛型类型推断、swtich、forkjoin、G1
- JDK8 lamda、stream、dateTime、
- JDK9 模块化

  

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

### 函数式编程

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

