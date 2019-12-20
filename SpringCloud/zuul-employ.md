# Zuul 的使用
---------------------

### 一、创建服务

- 选择 eureka 服务发现
- 选择 config client

### 二、启动类添加注解

```
@EnableZuulProxy
```

### 三、编写配置文件

```
1.将启动类修改为 bootstrap.yml

spring:
  application:
    name: api-getway
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG-SERVER
      profile: dev
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/
	  
2.将公共的配置内保存在配置中心

zuul:
  host:
    connect-timeout-millis: 10000
    socket-timeout-millis: 60000
  retryable: true
  routes:
    config-server:
      path: /myConfig/**   #将路由进行配置 这里是自定义路由前缀
      serviceId: config-server  #需要进行配置的路由 服务 ID
      sensitiveHeaders:  #配置 cookie 可以接收 如果不加这个参数 后端将无法接收到 cookie
ribbon:
  ReadTimeout: 2000

```

### 四、动态刷新配置

```
1.添加 bus 依赖

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

2.启动类文件中声明 zuul 的配置文件

@EnableDiscoveryClient
@EnableEurekaClient
@EnableZuulProxy
public class ApiGetwayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGetwayApplication.class, args);
    }

    @ConfigurationProperties("zuul")  //声明 zuul 的配置文件
    @RefreshScope  //设置动态刷新范围
    public ZuulProperties zuulProperties(){
        return new ZuulProperties();
    }
}
```
