# mcps

mcps基础框架

## 简介
该项目是基础框架， 提供通用类库、微服务框架及对中间件及第三方框架的封装，提供统一的接口
提供如缓存、消息队列、数据层访问等的接口

## 版本
2.1.0

## 设计

### 主要模块
- mcps-base: 通用类库， 提供工具类

- mcps-frame：框架类库， 提供第三方框架的封装  
![mcps-frame](img/mcps-frame.png)
    - mcps-framework-all: 所有代码的汇总
    - mcps-framework-core: 基础代码，微服务框架
    - mcps-framework-cache: 缓存API
    - mcps-framewrok-queue: 消息队列API
        - Queue接口
        - QueueListener注解
        - MessgeListener接口
    - mcps-framework-persistence: jpa、rsql, querydsl
    - mcps-framework-web: web
    - mcps-framework-trace: traceId跟踪
    - mcps-frame-redis: redis缓存实现
    - mcps-frame-kafka: kafka消息队列实现
        - Kafka队列及监听器实现
    - mcps-frame-rocketmq: rocketmq消息队列实现
        - rocketmq队列及监听器实现
- mcps-starter: spring boot启动器  
![mcps-starter](img/mcps-starter.png)

    - mcps-baffle-starter：挡板服务，实现真实数据和模拟数据之间切换

    - mcps-cache-starter：提供统一的缓存service接口

    - mcps-client-starter：非微服务项目的父POM

    - mcps-counter-starter：分布式计数器

    - mcps-doc-starter： swagger封装

    - mcps-dynamic-datasource-starter：动态数据源

    - mcps-idempotent-starter：幂等服务实现，目前是基于Redis的实现

    - mcps-jpa-starter：基于jpa的项目需要依赖

    - mcps-lock-starter：分布式锁，目前是基于Redis的实现

    - mcps-logger-starter：统一的日志格式处理，以及日志关键字脱敏,traceId跟踪

    - mcps-moniter-starter：监控功能实现，通过prometheus收集性能数据（metrics）并通过grafana展示

    - mcps-oauth2-starter：鉴权服务，实现访问权限的控制

    - mcps-queue-kafak-starter：kafka队列

    - mcps-queue-rocketmq-starter： rocketmq队列

    - mcps-root-starter：基础POM, 指定spring的依赖版本

        - spring cloud version：Hoxton.SR1
        - spring boot version：2.2.5.RELEASE

    - mcps-service-starter： 微服务starter, 微服务项目的父POM

    - mcps-starter-starter：启动器starter, spring boot启动器的父POM


### mcps-frame-core
实现一个微服务所需的核心类  
![mcps-framework-core](img/mcps-frame-core.png)  

- IDao： 数据层封装
- IService：简单CRUD服务接口的定义
    - AbstractService： 实现类
- IMapper: DTO与Entity的映射接口（MapStruct）
- AbstractController： 简单的CRUD的web接口
- DTO
    - BaseDTO： DTO基类
    - PageDTO: 分页DTO
- const
    - FrameworkCode： 通用的错误码定义
- entity
    - BaseEntiy 数据库实体的基类
- exception
    - FrameworkException： 框架运行时异常基类
- client
    - BaseClient: 客户端基类，通用异常处理
    - BaseFeignClient： FeignClient基类


## 微服务项目
微服务项目包括两种不同类型的服务：
* 后台项目 - 一般基于数据库，提供基础服务
* 聚合项目 - 一般与前台对接，屏蔽后台服务拓扑并提供接口安全配置

### 后台项目
  基于JPA实现
 - ${project}-model: DTO及接口定义，客户端与服务器端公用
 - ${project}-client： 提供客户端API
 - ${project}-service: 微服务
 - 示例  
![mcps-project](img/mcps-project.png)

 - 调用流程  
![mcps-project](img/mcps-demo.png)

### model实现  

 - DTO继承BasDTO

	```Java
	@Data
	@ApiModel("用户resp")
	public class User extends BaseDTO {    
		@ApiModelProperty("真实名字")    
		private String realname;    
		@ApiModelProperty("用户名")    
		private String username;
	}
	```

 - service继承IService, 定义业务接口

	```java
	public interface IUserService extends IService<User> {    
		/**     
		* 获取权限信息     
		* @param username     
		* @return     
		*/    
		User getAuthorityByUsername(String username);
	}
	```

### client实现

 - 继承BaseClient， 提供服务接口
    - 使用Client注解，指定FeignClient

	```java
	@Client(name="userClient",feignClient = FeignUserClient.class)
	public  class UserClient extends BaseClient<User> implements IUserService {    
		private final FeignUserClient feignUserClient;    
		public UserClient(FeignUserClient feignUserClient) {        
			this.feignUserClient = feignUserClient;    
		}    
		@Override    
		public User getAuthorityByUsername(String username) {        
			ResultMessage<User> resultMessage = null;        
			try{            
				resultMessage =  feignUserClient.getAuthorityByUsername(username);        
			} catch(Exception ex) {            
				handleException(ex);        
			}        
			return handleResultMessage(resultMessage, UserException.class);    
		}
	}
	```

 - 继承BaseFeignClient, Feign http rpc调用
    ```java
	@Api("用户客户端")
	@FeignClient(name = "mcps-demo-service",contextId ="user", path = "user")
	public interface FeignUserClient extends BaseFeignClient<User> {    
		@ApiOperation("用户授权信息查询")    
		@GetMapping({"/get/authority/{username}"})    
		ResultMessage<User> getAuthorityByUsername(@PathVariable String username);
	}
	```

### service实现
> spring boot项目接入

- pom依赖
	```
	<dependency>
	  <groupId>com.allinpay.mcps.starter</groupId>
	  <artifactId>mcps-service-starter</artifactId>
	  <version>2.0.0.RELEASE</version>
	</dependency>
	```

- starter依赖
使用JPA
	```java
		<dependency>
		  <groupId>com.allinpay.mcps.starter</groupId>
		  <artifactId>mcps-jpa-starter</artifactId>
		  <version>2.0.0.RELEASE</version>
		</dependency>
	```
 支持swagger
	```java
		<dependency>
		  <groupId>com.allinpay.mcps.starter</groupId>
		  <artifactId>mcps-doc-starter</artifactId>
		  <version>2.0.0.RELEASE</version>
		</dependency>
	```

​       根据项目的需求，增加相应的starter

- bootstrap.properties

  - resources里增加bootstrap配置nacos信息

- Controller 层实现
    - 继承AbstractController
    - 实现其他的Controller接口

	```java
	@Api("用户控制器")
  @RestController
  @RequestMapping("/user")
	public class UserController extends AbstractController<User> {    
		@Autowired    
		private UserService userService;    
		@Override    
		protected IService<User> getService() {        
			return userService;    
		}    
		@ApiOperation("用户授权信息查询")    
		@GetMapping({"/get/authority/{username}"})    
		public ResultMessage<User> getAuthorityByUsername(@PathVariable String username){     
			User user = userService.getAuthorityByUsername(username);        
			return ResultMessage.<User>builder().code(user != null ?
				   FrameworkCode.CODE_SUCCESS.getCode() : FrameworkCode.CODE_FAIL.getCode())                
           .msg(user != null ? "成功" : "获取当前用户信息失败").data(user)           
				   .build();    
		}
	}
	```

- Service层实现
    - 继承AbstractJpaService
        - 默认已实现单表的CRUD的操作
        - 实现两个抽象方法
            - 指定Dao层实现

			```java
			public interface UserDao extends JpaDao<UserEntity> {

				/**
				 * 查询手机号或者用户名
				 * @return
				 */
				UserEntity findFirstByUsernameOrPhone(String username, String phone);

				/**
				 * 查询手机号
				 * @param phone
				 * @return
				 */
				UserEntity findFirstByPhone(String phone);				
			}
			```

            - 指定Mapper实现
			```java
			@Mapper(componentModel="spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
			public interface UserMapper extends IMapper<User, UserEntity> {

			}
			```

    - 实现业务接口
	```java
	@Service
	public class UserServiceImpl extends AbstractJpaService<User, UserEntity> implements UserService {   
		private static final PasswordEncoder ENCODER = new BCryptPasswordEncoder();   

		@Autowired   
		UserDao userDao;   
		@Autowired   
		UserMapper userMapper;   

		@Override   
		protected JpaDao<UserEntity> getDao() {      
			return userDao;   
		}   

		@Override   
		protected IMapper<User, UserEntity> getMapper() {      
			return userMapper;  
		}  

		/**    
		* 查询用户权限    
		*    
		* @param username    
		* @return    
		*/   
		@Override   
		public User getAuthorityByUsername(String username) {      
			QUserEntity userEntity = QUserEntity.userEntity;      
			userDao.findAll(userEntity.email.between("11","222"));      
			userDao.findOne(userEntity.email.eq("11").and(userEntity.id.eq(1L)));      
			userDao.findAll(userEntity.email.eq("11"),userEntity.email.desc());      
			User user = get(1L).orElse(null);      
			userDao.findAll(ExpressionUtils.in(userEntity.address, Lists.newArrayList()));  
			return user;   
		}
	}
	```
- 数据层实现
    - Entity实现， 继承BaseEntity

  ```java
      @EqualsAndHashCode(callSuper = true)
      @Data
      @Entity
      @Table(name="tbl_sys_user")
      @Where(clause = "is_delete != 1")
      public class UserEntity extends JpaEntity {    
          @Column(columnDefinition = "varchar(32) comment '用户名'")   
          private String username;     
          @Column(columnDefinition = "varchar(64) comment '密码'")   
          private String password;   
          @Transient   
          private String newPassword;   
          @Column(columnDefinition = "varchar(32) comment '手机号'")   
          private String phone;   
          @Column(columnDefinition = "varchar(128) comment '邮箱'")   
          private String email;   
      }
  ```

- 引用Client
    - @EnableFeignClients(clients={UserFeignClient.class})
    - @EnableClients(clients={UserClient.class})

## 聚合项目
聚合项目一般实现底层微服务的转发和聚合，其主要是：
- 实现Controller层接口
- 注入底层微服务的client
- 授权与权限在聚合层实现

## 注意事项
### Service层事务
- Transcational事务注解不要写在类上，注解在方法上
- Transcational事务方法要防止长事务
  - 事务方法中不能有远程的调用
  - 一般的设计范式: 事务 -> 远程 -> 事务及冲正

### 缓存
- Dao层缓存
  - 通过Cacheable注解实现
  - cache key需符合规范
  - 示范：@Cacheable(value = "xtys:cache:user:mobile", key = "#p0")

### 异常处理
- 状态码
  - 参考错误码规范：http://confluence.allinpaycard.cn/pages/viewpage.action?pageId=1671764
- 统一异常处理
  - 业务一般定义一个业务异常，并通过错误码来区分不同的异常
```Java
/**
* 定义错误码
*/
public enum BatchCode {
    /**
     * 批处理任务已经存在
     */
    CODE_BATCH_ALREADY_EXISTS("BATCH001", "批处理任务已经存在");
  }

  /**
  * 定义业务异常
  */
public class BatchJobException extends FrameworkException {
    public BatchJobException(String code, String msg) {
        super(code, msg);
    }

    public BatchJobException(String code, String msg, String detail) {
        super(code, msg, detail);
    }
}

/**
* 根据具体情况，抛出业务异常，设置正确的错误码
*/
if(batchJobByJobName.isPresent()){
  throw new BatchJobException(CODE_BATCH_ALREADY_EXISTS.getCode(),
          CODE_BATCH_ALREADY_EXISTS.getDesc(),
          job.getJobName() + "已存在");
}

- Controller通过GlobalExceptionHandler处理异常， 生成ResultMessage返回客户端
- 客户端通过Code来重构业务异常

```
### 跨服务追踪链(traceid)

#### 原理

   [Slf4j MDC( Mapped Diagnostic Contexts )机制](https://www.jianshu.com/p/1dea7479eb07 "Slf4j MDC机制")

     1.请求入口，针对web servlet 通过拦截器，获取请求，如果头中无 traceId,则增加
     2.服务内部，针对异步线程池，在执行run方法时，添加traceId
     3.其他服务调用，针对Feign调用，在请求头部增加traceId


#### 客户端使用步骤

- 1.pom引用
```
    <!--pom引用-->
    <dependency>
        <groupId>com.allinpay.mcps.starter</groupId>
        <artifactId>mcps-logger-starter</artifactId>
        <version>2.0.0.RELEASE</version>
    </dependency>
```

- 2.异步线程池使用
 @Async("traceTaskExecutor")

```java
    // 示例
    @Async("traceTaskExecutor")
    public void doTaskOne() throws Exception {
        log.info("开始做任务一");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        log.info(TraceContext.getTraceId() + "完成任务一，耗时：" + (end - start) + "毫秒");
    }
```

## 参数配置
服务的默认配置位于nacos的application.yml中

### feignClient配置
参考配置类：FeignClientProperties
- 配置文件配置

```yaml
feign:
  client:
    config:
      default:   # 默认配置，名称可以通过feign.client.default-config=my-config指定
        connectTimeout: 10000
        readTimeout: 10000
  client：
    config：
      user： // 特定FiegnClient配置，名称通过FeignClient的name指定
        connectionTimeout： 6000  
```

- java 配置
可以通过FeignClient指定FeignClientsConfiguration来自定义配置

### 重试
- 默认情况下Feign的本身的重试是关闭的，但spring cloud openfeign默认是使用Ribbon实现负载均衡和重试机制的。
- ribbon默认配置

```yml
ribbon:
  # 等待请求响应的超时时间. 单位：ms
  ReadTimeout: 5000
  # 连接超时时间. 单位：ms
  ConnectTimeout: 1000
  # 是否对所有请求进行失败重试, 设置为 false, 让feign只针对Get请求进行重试. 
  # 如果Post方法支持幂等操作，可以设置true
  OkToRetryOnAllOperations: false


  # 关闭重试
  # Max number of retries on the same server (excluding the first try)
  MaxAutoRetries: 0
  # Max number of next servers to retry (excluding the first server)
  MaxAutoRetriesNextServer: 0
```

- 不同服务特定配置
```
app1: #针对app1的ribbon配置
  ribbon:
    ReadTimeout: 10000
    ...
```

> 参考代码
> FeignLoadBalancer
> RequestSpecificRetryHandler

### 熔断配置
```

```

## 贡献者
* 谢强 (xieqiang1@allinpay.com)
