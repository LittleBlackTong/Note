# hystrix 服务容错保护

### 背景

> 当服务调用 从 A服务 -> B服务 -> C服务  如果 C服务崩溃 那么那么 B服务调用C服务时会同步等待最后系统资源耗尽,A服务调用B服务最后A服务也会资源耗尽最终导致整个项目崩溃  雪崩效应

### 依赖隔离

> hystrix 线程池隔离 为hyxstrixcommand 独立开启一个线程,自动实现了依赖隔离

### 服务熔断

> circuitBreaker 断路器 

```
@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000"),
@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")

circuitBreaker.sleepWindowInMilliseconds 当服务发生熔断会有一个窗口时间,这个时间内默认将服务降级,过了窗口时间后,将一部分请求会回复到正常的流程,如果继续有问题会继续短路开启, 窗口时间回复
circuitBreaker.requestVolumeThreshold 设置滚动窗口时间中最小请求数
circuitBreaker.errorThresholdPercentage 短路错误百分比条件 超过百分比请求数就断路器开启

```

### 项目中使用 hystrix

```
@HystrixCommand(fallbackMethod = "getHelloFallBack")  //当usercenter 服务down 的时候会触发服务降级 返回 getHelloFallBack 方法
public String getHello(Integer test) {
	return restTemplate.getForObject("http://usercenter/hello", String.class);
}

private String getHelloFallBack() {
        return "userCenter server down please wait~";
}

// 也可以设置默认的服务降级时执行的方式,在实现类上使用这个注解 会执行默认的服务降级方法

@DefaultProperties(defaultFallback = "defaultFallBack")
public class UserServiceImpl implements UserService {

	    private String defaultFallBack() {
			return "default server down please wait~";
    }
}
```

### 配置服务超时时间

1. 项目添加注解
```
//可以在  HystrixCommand 注解内的包 com.netflix.hystrix.contrib.javanica.annotation; 找到对应的配置名称
@HystrixCommand(commandProperties = {
	@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000")
}

```
2. yml 配置文件中配置 注意点: 如果要让默认配置生效 一定要在对应方法上添加 @Hystrixcommand 注解 不然不会生效

```
hystrix:
  command:
    default:  #设置服务默认的超时时间
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000
    getHello: # 设置具体的某个方法的超时时间
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000
```

### 服务熔断

> 断路器机制,当服务出现降级之后,可以设置默认的一段时间内使服务熔断 避免出现更多的错误

```
@HystrixCommand(commandProperties = {
	@HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
	@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
	@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000"),
	@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")
})
```

### 设置 feign-hystrix

```
/**
 * @author littleblack
 */
@FeignClient(
        name = "USERCENTER",
        fallback = UserCenterFeignClient.UserCenterFeignClientFallBack.class)
public interface UserCenterFeignClient {

    /**
     * 用户中心 hello 接口
     *
     * @return prompt
     */
    @GetMapping("/hello")
    String hello();

    @Component  //一定要加commponent 注解 使spring可以创建 注意注解扫描需要 扫描到 client 的包
    public static class UserCenterFeignClientFallBack implements UserCenterFeignClient {

        @Override
        public String hello() {
            return "usercenter hello is down wait a moment~";
        }
    }
}
```

### feign-hystrix 配置
```
feign:  #需要将 feign hystrix 开启 不然 hystrix 会失效
  hystrix:
    enabled: true
  client: # 需要将 client 的超时时间设置 不然会出现readTimeout 错误
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
```

### feign 开启 debug 日志

```
/**
 * @author littleblack TongTong
 * @create 2020-05-17 6:27 下午
 **/
//创建feign client 的配置类
@Configuration
public class FeignClientConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}


logging: # 为了在链路追踪时开启日志
  level:
    com.illusion.spring.feignclient: debug
```
