# Spring Cloud Gateway

## 断言

根据断言匹配路由

### 内建断言

- 时间断言
  - 之前
  - 之后
  - 之间
- Cookie
- 请求头
- 主机
- 请求方法
- 路径
- 查询参数
- IP
- 权重



## 过滤器

### GlobalFilter

#### 前置

```java
public GlobalFilter customGlobalFilter() {
    return (exchange, chain) -> exchange.getPrincipal()        
        .map(Principal::getName)        
        .defaultIfEmpty("Default User")        
        .map(userName -> {          
            //adds header to proxied request*          
            exchange.getRequest().mutate().header("CUSTOM-REQUEST-HEADER", userName).build();          			  
            return exchange;        
        })        
        .flatMap(chain::filter); 
}
```



#### 后置

```java
public GlobalFilter customGlobalPostFilter() {
    return (exchange, chain) -> chain.filter(exchange)        
        .then(Mono.just(exchange))        
        .map(serverWebExchange -> {
            //adds header to response*       
            serverWebExchange.getResponse().getHeaders().set("CUSTOM-RESPONSE-HEADER",              HttpStatus.OK.equals(serverWebExchange.getResponse().getStatusCode()) ? "It worked": "It did not work");  
            return serverWebExchange;        
        })        
        .then(); 
}
```



### AdaptCachedBodyGlobalFilter

#### 全局激活

```java
//让每一个路径都做body Cache，这样重试有Body的请求的时候，重试的请求不会没有body，
//因为原始body是一次性的基于netty的FluxReceive
gatewayProperties.getRoutes().forEach(routeDefinition -> {
            EnableBodyCachingEvent enableBodyCachingEvent = new EnableBodyCachingEvent(new Object(), routeDefinition.getId());
            adaptCachedBodyGlobalFilter.onApplicationEvent(enableBodyCachingEvent);
        });
```

#### 路由激活

```java
builder.routes().route("modifybody_route", r -> r.host("*.modifybody.org")
						.filters(f -> f.modifyRequestBody(String.class, String.class, MediaType.APPLICATION_JSON_VALUE,
			                    (exchange, s) -> Mono.just(new String(s.toUpperCase()))))
						.uri("http://baidu.com"))
```

#### 获取Body

```java
DataBuffer requestBody = (DataBuffer)exchange.getAttributeOrDefault(CACHED_REQUEST_BODY_ATTR, null);
String body = StandardCharsets.UTF_8.decode(requestBody.asByteBuffer()).toString();
//操作body...

DataBuffer bodyDataBuffer = stringBuffer(body);
Flux<DataBuffer> bodyFlux = Flux.just(bodyDataBuffer);
ServerHttpRequest request = new ServerHttpRequestDecorator(serverHttpRequest){
	@Override
	public Flux<DataBuffer> getBody() {
		return bodyFlux;
	}
};//封装我们的request
return chain.filter(exchange.mutate().request(request).build());
```

body读取后，要重新构造

### GatewayFilterFactroy

#### 命名规范

以GatewayFilterFactory结尾

#### Order

工厂创建出来的过滤器是没有指定order的，会被默认设置为是0，

配置在yml文件中，则按照它书写的顺序来执行.

如果要指定order，需实现一个内部GatewayFilter

#### 前置

```java
public class PreGatewayFilterFactory extends AbstractGatewayFilterFactory<PreGatewayFilterFactory.Config> {

    public PreGatewayFilterFactory() {
        super(Config.class);
    }
    
    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            //If you want to build a "pre" filter you need to manipulate the
            //request before calling chain.filter
            ServerHttpRequest.Builder builder = exchange.getRequest().mutate();
            //use builder to manipulate the request
            return chain.filter(exchange.mutate().request(builder.build()).build());
        };
    }
    
    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```

#### 后置

```java
public class PostGatewayFilterFactory extends AbstractGatewayFilterFactory<PostGatewayFilterFactory.Config> {

    public PostGatewayFilterFactory() {
        super(Config.class);
    }
    
    @Override
    public GatewayFilter apply(Config config) {
        // grab configuration from Config object
        return (exchange, chain) -> {
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                ServerHttpResponse response = exchange.getResponse();
                //Manipulate the response in some way
            }));
        };
    }
    
    public static class Config {
        //Put the configuration properties for your filter here
    }

}
```



#### 样例

```java
@Component
public class LoginCaptchaFilterGatewayFilterFactory extends AbstractGatewayFilterFactory<Config> {

    private final RedisUtil redisUtil;
    
    public LoginCaptchaFilterGatewayFilterFactory(RedisUtil redisUtil) {
        super(Config.class);
        this.redisUtil = redisUtil;
    }

    @Override
    public GatewayFilter apply(Config config) {
        //return new InnerGatewayFilter(config);
        return (exchange, chain) -> {
            ServerHttpRequest serverHttpRequest = exchange.getRequest();
            URI uri = serverHttpRequest.getURI();
            // 不是登录请求，直接向下执行
            if(!config.enabled) {
                return chain.filter(exchange);
            }
            if (!StringUtils.containsIgnoreCase(uri.getPath(), config.authUrl)) {
                return chain.filter(exchange);
            }
            if (HttpMethod.POST.matches(serverHttpRequest.getMethodValue())) {
                String bodyStr = ServletHttpHelper.resolveBodyFromRequest(serverHttpRequest);
                try {
                    JSONObject bodyJson = JSONObject.parseObject(bodyStr);
                    String username = String.valueOf(bodyJson.get("username"));
                    String password = String.valueOf(bodyJson.get("password"));
                    String code = String.valueOf(bodyJson.get("code"));
                    String uuid = String.valueOf(bodyJson.get("uuid"));
                    // 登陆校验
                    loginCheckPre(username, password, code, uuid);
                } catch (Exception e) {
                    ServerHttpResponse response = exchange.getResponse();
                    response.setStatusCode(HttpStatus.OK);
                    response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
                    AjaxResult ajaxResult = AjaxResult.error(HttpStatus.INTERNAL_SERVER_ERROR.value(), e.getMessage());
                    String msg = JSON.toJSONString(ajaxResult);
                    DataBuffer bodyDataBuffer = response.bufferFactory().wrap(msg.getBytes());
                    return response.writeWith(Mono.just(bodyDataBuffer));
                }
                ServerHttpRequest request = serverHttpRequest.mutate().uri(uri).build();
                DataBuffer bodyDataBuffer = ServletHttpHelper.transferBodyStrToDataBuffer(bodyStr);
                Flux<DataBuffer> bodyFlux = Flux.just(bodyDataBuffer);
                request = new ServerHttpRequestDecorator(request) {
                    @Override
                    public Flux<DataBuffer> getBody() {
                        return bodyFlux;
                    }
                };
                return chain.filter(exchange.mutate().request(request).build());
            }
            return chain.filter(exchange);
        };
    }
    
    private static class InnerGatewayFilter implements GatewayFilter, Ordered {
        public InnerGatewayFilter(Config config) { ... }
         @Override
        public int getOrder() {
            return -100;
        }
    }
    
    
     /**
     * 自定义的config类，用来设置传入的参数
     */
    @Data
    public static class Config {
        // 是否启用
        private boolean enabled;
        // 白名单
        private String authUrl;
    }
```

```yml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 表明gateway开启服务注册和发现的功能，并且spring cloud gateway自动根据服务发现为每一个服务创建了一个router
          lower-case-service-id: true  # 服务名小写
      # 路由(如果使用动态路由方式，不要在配置文件中配置路由）
      routes:
        # 认证中心
        - id: nevims-app-auth
          uri: lb://nevims-auth-service
          predicates:
            - Path=/nevims/api-auth/**
          filters:
            # 验证码处理
            - StripPrefix=2
            - name: LoginCaptchaFilter
              args:
                enabled: true
                authUrl: '/auth/login'
```

## 路由

### RouteDefinitionRepository

### nacos实现



```java
public class NacosRouteDefinitionRepository implements RouteDefinitionRepository {
    private static final String SCG_DATA_ID = "scg-routes";
    private static final String SCG_GROUP_ID = "SCG_GATEWAY";

    private ApplicationEventPublisher publisher;

    private NacosConfigProperties nacosConfigProperties;

    public NacosRouteDefinitionRepository(ApplicationEventPublisher publisher, NacosConfigProperties nacosConfigProperties) {
        this.publisher = publisher;
        this.nacosConfigProperties = nacosConfigProperties;
        addListener();
    }

    @Override
    public Flux<RouteDefinition> getRouteDefinitions() {
        try {
            String content = nacosConfigProperties.configServiceInstance().getConfig(SCG_DATA_ID, SCG_GROUP_ID,5000);
            List<RouteDefinition> routeDefinitions = getRouteDefinitions(content);
            return Flux.fromIterable(routeDefinitions);
        } catch (NacosException e) {
            log.error("getRouteDefinitions by nacos error", e);
        }
        return Flux.fromIterable(CollUtil.newArrayList());
    }
    
     private void addListener() {
        try {
            nacosConfigProperties.configServiceInstance().addListener(SCG_DATA_ID, SCG_GROUP_ID, new Listener() {
                @Override
                public Executor getExecutor() {
                    return null;
                }

                @Override
                public void receiveConfigInfo(String configInfo) {
                    publisher.publishEvent(new RefreshRoutesEvent(this));
                }
            });
        } catch (NacosException e) {
            log.error("nacos-addListener-error", e);
        }
    }
}
```

### redis实现



### Actuator EndPoints

**management.endpoint.gateway.enabled**=true # default value **management.endpoints.web.exposure.include**=gateway

path prefix: /actuator/gateway/

| Path          | HttpMetho | Description                                                  |
| ------------- | --------- | ------------------------------------------------------------ |
| globlefilters | Get       | Displays the list of global filters applied to the routes.   |
| routefilters  | Get       | Displays the list of `GatewayFilter` factories applied to a particular route. |
| refresh       | Post      | Clears the routes cache.                                     |
| routes        | Get       | Displays the list of routes defined in the gateway.          |
| routes/{id}   | Get       | Displays information about a particular route.               |
| routes/{id}   | Post      | Adds a new route to the gateway.                             |
| routes/{id}   | Delete    | Removes an existing route from the gateway.                  |



## 超时配置

### 全局配置

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

### 特定路由配置

设置元数据 

yml

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: per_route_timeouts
          uri: https://example.org
          predicates:
            - name: Path
              args:
                pattern: /delay/{timeout}
          metadata:
            response-timeout: 200
            connect-timeout: 200
```

java

```java
routeBuilder.routes()
    .route("test1", r -> {
        return r.host("*.somehost.org").and().path("/somepath")
            .filters(f -> f.addRequestHeader("header1", "header-value-1"))
            .uri("http://someuri")
            .metadata(RESPONSE_TIMEOUT_ATTR, 200)
            .metadata(CONNECT_TIMEOUT_ATTR, 200);
```



## 限流

Sentinel限流

限流资源 @sentinelResource

### 融断

#### 限流熔断

- 指定blockhandler

#### 异常熔断

- 指定fallback

```java
  @GetMapping("/testA")
  @SentinelResource(value = "testA", 
                    blockHandlerClass = CustomerCircuitBreaker.class, blockHandler = "blocHandler", 
                    fallbackClass = {CustomerCircuitBreaker.class}, fallback = "resultException")
  public CommonResult testA() {
        int a = 1 / 0;
        return new CommonResult(200, "testA");
 }

@Slf4j
public class CustomerCircuitBreaker {
    public static CommonResult resultException(Throwable throwable) {
        return new CommonResult(500,"运行异常回调处理");
    }

     public static CommonResult<String> blocHandler(BlockException blockException) {
        return new CommonResult(500,"自定义block1");
    }

}
```

### 规则持久化

- 引入sentinel-datasource-nacos
- 配置数据源

```yml
sentinel:
      transport:
        dashboard: 127.0.0.1:8080
        port: 8719 #默认就是8719 如果被占用默认+1
      datasource:
        ds1:
          nacos:
            server-addr: 192.168.10.37:18848
            dataId: ${spring.application.name}
            groupId: DEV
            namespace: 8622d428-0496-4a09-b178-da3cfc736055
            data-type: json
            rule-type: flow
```

- nacos配置

  ```json
  [
      {
          "resource":"testB",  // 资源名称
          "limitAPP":"default", //来源应用
          "grade":1, // 阈值类型, 0表示线程数,1表示QPS
          "count":1, //单机阈值
          "strategy":0, //流控模式,0表示直接,1表示关联,2表示链路
          "controlBehavior":0, // 流控效果,0表示快速失败,1表示WarmUp,2表示排队等待
          "clusterMode":false //是否集群
      }
  
  ]
  ```

  

## 问题解答

- DataBufferLimitException: Exceeded limit on max bytes to buffer

  https://github.com/spring-cloud/spring-cloud-gateway/issues/1658

- 

## 网关设计

### 目的

支持以下功能：

1. 动态路由
2. 基于路由的验签和加密过滤器

### 架构图

![image-20210104101028837](C:\Users\richardxie\AppData\Roaming\Typora\typora-user-images\image-20210104101028837.png)

### 设计

#### 功能

1. 动态网关， Redis支持， nacos支持
2. 定制化验签方式
   1. 通过Path断言，确定不同的SignatureGatewayFilter配置
   2. SignatureGatewayFilter通过配置使用不同的验签方式

#### 流程

 	1. nacos配置Path断言，确定验签方式
 	2. SigatureFilter根据配置，调用不同的验证类验证签名
 	  	1. 通过请求参数来验签
 	  	2. 通过请求Body来验签
 	3. Body缓存

#### 设计

##### 主要类

1. NacosRouteDefinitionRepository nacos动态路由
2. SignatureGatewayFilter 签名过滤器
3. SignatureConfig 

## 参考

Post请求验签

https://www.it610.com/article/1282318695766441984.htm

https://gitee.com/zlt2000/microservices-platform/tree/master/zlt-gateway