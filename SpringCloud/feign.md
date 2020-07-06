# Feign 使用
---------------------
> - 一、声明式REST客户端（伪RPC）
> - 二、采用基于接口注解方式

### 使用方式：
1. 引入依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
	<optional>true</optional>
</dependency>
```
2. 启动类添加注解

```
@EnableFeignClients("com.tongtong.*")
```

3. feignclient 配置

```
@Configuration
public class FeignClientConfig {
    @Bean
    Logger.Level feignLoggerLevel() { //设置日志等级
        return Logger.Level.FULL;
    }
}
```

3. 声明调用接口

```
@FeignClient(name = "eureka-client") //其他服务名称
public interface FeinClientMethod {
    @GetMapping("/msg") //其他服务接口名称
    String getMsg();
}
```

> feignClient 在实际使用时需要和 hystrix 一起使用当,接口出现故障,通过 hystrix 进行熔断处理防止服务雪崩

### feignClient 对比 Ribbon+restTemplate 

> Ribbon 是客户端负载均衡器,它通过读取服务列表,默认通过轮训的方式来进行客户端负载均衡
> feignClient 是在 Ribbon 的基础上进行操作的
> feignClient 采用接口+注解的方式简化了消费服务时的复杂操作
> Ribbon 和 Resttemplate 则需要开发者手动编写服务请求,操作相对复杂
