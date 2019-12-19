# Feign 使用
---------------------
> - 一、声明式REST客户端（伪RPC）
  - 二、采用基于接口注解方式

### 使用方式：
1.引入依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
	<optional>true</optional>
</dependency>
```
2.启动类添加注解

```
@EnableFeignClients("com.tongtong.*")
```
3.声明调用接口

```
@FeignClient(name = "eureka-client") //其他服务名称
public interface FeinClientMethod {
    @GetMapping("/msg") //其他服务接口名称
    String getMsg();
}
```
