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

2. 尝试从缓存中加载单例

   检查缓存中或者实例工厂中是否有对应的实例。Spring创建bean的原则是不等bean创建完成就会见创建bean的ObjectFactory提早加入缓存。这样避免循环依赖。

   Object sharedInstance = getSingletom(beanName);

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

   RootBeanDefiniton

7. 寻找依赖

   Bean如果有dependOn，先要加载依赖的Bean

8. 针对不同的scope进行Bean的创建

   singleton, prototype, request等不同的scope，进行Bean初始换

9. 类型转换

   根据接口的requiredType进行类型转换



