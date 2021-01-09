# Java

## Java泛型

### 类型擦除

- java伪泛型，编译器语法糖， 运行时无泛型信息，无缝使用旧代码

- class类文件的Signature属性

  > Signature 属性是可选的定长属性，位于 ClassFile， field_info 或 method_info结构的属性表中。
  >
  > 在 Java 语言中，任何类、 接口、 初始化方法或成员的泛型签名如果包含了类型变量（ Type Variables） 或参数化类型（ Parameterized Types），则 Signature 属性会为它记录泛型签名信息。

- 获取类型信息

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

  

