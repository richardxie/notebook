

# Springboot

## 概念

spring boot提供了开箱即用的快速开发的能力。

- 特点

  - 独立运行，内嵌Web容器，fatjar

  - Starter， 简化配置，自动配置（约定先于配置）

  - 常用的非功能级特性： 安全、监控数据、健康检查和外部配置

  - 无代码生成和 xml 配置



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