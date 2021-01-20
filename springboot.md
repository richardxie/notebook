

# Springboot

## 概念

spring boot提供了开箱即用的快速开发的能力。

- 特点

  - 独立运行，内嵌Web容器，fatjar

  - Starter， 简化配置，自动配置（约定先于配置）

  - 常用的非功能级特性： 安全、监控数据、健康检查和外部配置

  - 无代码生成和 xml 配置

## Cache

### 缓存的问题

- **缓存穿透预防及优化**

  缓存穿透是指查询一个根本不存在的数据，缓存层和存储层都不会命中，如果从存储层查不到数据不写入缓存层，**每次请求都要到存储层去查询，失去了缓存保护后端存储的意义**，引发缓存穿透。

  解决方案

  | 方式       | 场景                       | 成本                                           |
  | ---------- | -------------------------- | ---------------------------------------------- |
  | 缓存空对象 | 1.命中不高<br />2.实时性高 | 1. 简单<br />2. 更多缓存空间<br />3.数据不一致 |
  | 布隆过滤器 | 1.命中不高<br />2.实时性低 | 1. 复杂<br />2. 缓存空间少<br />               |

  

- **缓存雪崩**

  缓存存层由于某些原因整体不能提供服务，于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况

  解决方案

  保证缓存高可用

  依赖隔离组件为后端限流并降级

  预演缓存宕掉后，应用层做预案

  

  

- **缓存热点**

  缓存 + 过期时间的策略既可以加速数据读写，又保证数据的定期更新，这种模式基本能够满足绝大部分需求。但是有两个问题如果同时出现，可能就会对应用造成致命的危害：

  1. 热点 key
  2. 重建缓存不能在短时间完成

  在缓存失效的瞬间，有大量线程来重建缓存 ( 如下图)，造成后端负载加大，甚至可能会让应用崩溃。

  解决方案：

  全局锁， 此方法只允许一个线程重建缓存，其他线程等待重建缓存的线程执行完，重新从缓存获取数据即可

  不过期：不设置过期时间，或后台在key过期时自动重建key

## 事务

### 本地事务

- ACID

  **A（Atomic）**：原子性

  **C（Consistency）**：一致性

  **I（Isolation）**：隔离性

  **D（Durability）**：持久性

### 分布式事务

- CAP

  **C（Consistency）** ：一致性

  **A ( Availability )** : 可用性

  **P( Partition tolerance )**: 分区容忍性

- BASE

  **Basically Available**（基本可用）分布式系统在出现故障时，允许损失部分可用功能，保证核心功能可用

  **Soft state**（软状态）: 中间状态，这个状态不影响系统可用性，如订单的"支付中"、“数据同步中”等状态，待数据最终一致后状态改为“成功”状态。

  **Eventually consistent** （最终一致性）

  BASE 理论是对 CAP 中 AP 的一个扩展，通过牺牲强一致性来获得可用性，当出现故障允许部分不可用但要保证核心功能可用，允许数据在一段时间内是不一致的，但最终达到一致状态。满足BASE理论的事务，我们称之为“**柔性事务**”。

  

  |            | 2PC      | TCC        | 可靠消息   | **最大努力通知** |
  | ---------- | -------- | ---------- | ---------- | ---------------- |
  | 一致性     | 强一致性 | 最终一致性 | 最终一致性 | 最终一致性       |
  | 吞吐量     | 低       | 中         | 高         | 高               |
  | 实现复杂度 | 易       | 难         | 中         | 易               |

  

- https://www.cnblogs.com/dyzcs/p/13780668.html

## Spring Boot Docker

- dockerfile

```dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
#Shell是1号进程， 可读取环境变量
# ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
#java是1号进程，可读取环境变量
#ENTRYPOINT [ "sh", "-c", "exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
# ENTRYPOINT ["run.sh"]
```

- run.sh

```shell
#!/bin/sh
exec java ${JAVA_OPTS} -jar /app.jar ${@}
```



### timeZone & Fonts

```dockerfile
#拷贝字体文件
COPY ./simhei.ttf /usr/share/fonts/simhei.ttf  
#设置字符集
ENV LANG en_US.UTF-8 
#安装字体软件，完成字体配置
RUN apk add --update ttf-dejavu fontconfig && rm -rf /var/cache/apk/*

#时区
RUN apk --no-cache add tzdata  && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone 
```



### Smaller Image

- The `alpine` images are smaller than the standard `openjdk` library images from [Dockerhub](https://hub.docker.com/_/openjdk/).

- spring layer jar

  ```dockerfile
  #  build the fat jar, then unpack it 
  ROM openjdk:8-jdk-alpine as build
  WORKDIR /workspace/app
  
  COPY mvnw .
  COPY .mvn .mvn
  COPY pom.xml .
  COPY src src
  
  RUN ./mvnw install -DskipTests
  RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)
  
  # layer image
  FROM openjdk:8-jdk-alpine
  VOLUME /tmp
  ARG DEPENDENCY=/workspace/app/target/dependency
  COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
  COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
  COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app
  ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]
  ```

  


## Spring on K8s

1. Add readiness and liveness probes
2. Wait for container lifecycle processes to finish
3. Enable graceful shutdown


- k8s/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: gs-spring-boot-k8s
  name: gs-spring-boot-k8s
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gs-spring-boot-k8s
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: gs-spring-boot-k8s
     spec:
       containers:
       - image: spring-k8s/gs-spring-boot-k8s:snapshot
         name: gs-spring-boot-k8s
         resources: {}
         livenessProbe:
           httpGet:
             path: /actuator/health/liveness
             port: 8080
         readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
         lifecycle:
           preStop:
             exec:
               command: ["sh", "-c", "sleep 10"]  
         terminationGracePeriodSeconds: 60
         volumeMounts:
            - name: config-volume
              mountPath: /workspace/config
        volumes:
          - name: config-volume
          configMap:
            name: gs-spring-boot-k8s
status: {}
```

- k8s/application.properties

```properties
server.shutdown=graceful
management.endpoints.web.exposure.include=*
```

```shell
 kubectl create configmap gs-spring-boot-k8s --from-file=./k8s/application.properties
 kubectl get configmap gs-spring-boot-k8s -o yaml
```

https://help.aliyun.com/document_detail/132165.html