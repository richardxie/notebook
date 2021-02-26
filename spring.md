# Spring

## framework
### Bean
- 常见标签
    - id
        id标签是bean的唯一标识，IoC容器中bean的id标签不能重复，否则报错
        
    - name
      name标签，可以是分隔符分割的字符串，作为bean的别名列表，当ID不指定是，第一个name作为Bean的注册的ID（beanName）
      
    - class
      class属性是bean常用属性，为bean的全限定类名

    - factory-method

    - factory-bean

    - init-method

    - destory-method

    - scope

        

    - autowire

    ``` xml
    <bean id="user" class="com.demo.User" />
    ```

    - 其他
    - 

### 循环依赖

两个或多个Bean相互之间持有对方，包括构造器循环依赖和setter循环依赖。

- 构造器循环依赖

  通过构造器注入构成的循环依赖，此依赖无法解决，只能抛出BeanCurrentlyInCreationException异常。

  ```java
  @Component
  public class BeanA {
  	private final BeanB beanB;
  	public BeanA(BeanB b) {
  		this.beanB = b;
  	}
  	
  	public void a() {
  		System.out.println("BeanA")
  	}
  }
  
  @Component
  public class BeanB {
  	private final BeanB beanC;
  	public BeanB(BeanC c) {
  		this.beanC = c;
  	}
  	
  	public void b() {
  		System.out.println("BeanB")
  	}
  }
  @Component
  public class BeanC{
      private final BeanA beanA;
  	public BeanC(BeanA a) {
  		this.beanA = a;
  	}
  }
  ```

  



### BeanFactory

#### 加载Bean

```java
@Override
public Object getBean(String name) throws BeansException {
   return doGetBean(name, null, null, false);
}
```

1. 转换对应的BeanName

   别名转换， FactoryBean

   ```java
   final String beanName = transformedBeanName(name);
   ```

2. 尝试从缓存中加载单例

   检查缓存中或者实例工厂中是否有对应的实例。Spring创建bean的原则是不等bean创建完成就会见创建bean的ObjectFactory提早加入缓存。这样避免循环依赖。

   Object sharedInstance = getSingletom(beanName);

   ```java
   protected Object getSingleton(String beanName, boolean allowEarlyReference) {
       Object singletonObject = this.singletonObjects.get(beanName);
       if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
           synchronized (this.singletonObjects) {
               singletonObject = this.earlySingletonObjects.get(beanName);
               if (singletonObject == null && allowEarlyReference) {
                   ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                   if (singletonFactory != null) {
                       singletonObject = singletonFactory.getObject();
                       this.earlySingletonObjects.put(beanName, singletonObject);
                       this.singletonFactories.remove(beanName);
                   }
               }
           }
       }
       return (singletonObject != NULL_OBJECT ? singletonObject : null);
   }
   ```

   三级缓存:

   - **singletonObjects**，单例缓存，存储已经实例化完成的单例。bean name --> bean instance

   - **singletonFactories**，生产单例的工厂的缓存，存储工厂。bean name --> ObjectFactory 

   - **earlySingletonObjects**，提前暴露的单例缓存，这时候的单例刚刚创建完，但还会注入依赖。bean name --> bean instance

3. bean实例化（缓存中）

   bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);

4. 原型模式的依赖检查

   ```
   if (isPrototypeCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(beanName);
   }
   ```

5. 检测parentBeanFactory

   递归父类工厂加载Bean

6. 将存储XML配置文件的GernericBeanDefinition转换为RootBeanDefinition

   **GenericBeanDefinition**记录了一些当前类声明的属性或构造参数，但是对于父类只用了一个 `parentName` 来记录, 需要完整的类信息（**RootBeanDefiniton**)

7. 寻找依赖

   Bean如果有dependOn，先要加载依赖的Bean

8. 针对不同的scope进行Bean的创建

   singleton, prototype, request等不同的scope，进行Bean初始换

   - **使用工厂方法创建**，`instantiateUsingFactoryMethod` 。
   - **使用有参构造函数创建**，`autowireConstructor`。
   - **使用无参构造函数创建**，`instantiateBean`

9. 类型转换

   根据接口的requiredType进行类型转换
   
10. 注入属性

    ```java
    protected void populateBean ... {
        PropertyValues pvs = mbd.getPropertyValues();
        
        ...
        // InstantiationAwareBeanPostProcessor 前处理
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    continueWithPropertyPopulation = false;
                    break;
                }
            }
        }
        ...
        
        // 根据名称注入
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
    
        // 根据类型注入
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
    
        ... 
        // InstantiationAwareBeanPostProcessor 后处理
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                pvs = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                if (pvs == null) {
                    return;
                }
            }
        }
        
        ...
        
        // 应用属性值
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
    ```



优雅下线

- 首先关闭 socket 监听，等待正在处理的所有请求完成：具体可见 WebServerGracefulShutdownLifecycle，通过 getPhase 返回最大值，达到早于 WEB 容器关闭执行的目的；
- 然后触发 WEB 容器关闭：具体可见 WebServerStartStopLifecycle。

```yml
# 开启优雅关闭 
server: 
  shutdown: graceful 
# 关闭的缓冲时间
spring: 
  lifecycle: 
    timeout-per-shutdown-phase: 10s
    
# http://service/actuator/shutdown 
management:
  endpoint:
    shutdown:
      enabled: shutdown

```

k8s

https://help.aliyun.com/document_detail/132165.html

https://www.jianshu.com/p/9ea61d204559

