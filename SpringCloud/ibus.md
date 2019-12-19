# SpringCloud IBUS 实现

------------------

### 一、实现自动刷新

IBUS 刷新配置流程：
![ibus 刷新流程图](https://github.com/LittleBlackMann/Note/blob/master/Image/config-server-ibus.jpg?raw=true "ibus 刷新流程图")

### 二、IBUS 自动创建MQ队列

注意:rabbitmq 如果不配置则采用默认配置
```
如果需要更改则添加配置
spting:
    rabbitmq:
        host:
        port:
        username:
        password:
```
### 三、config-server 端配置
1.添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
2.将/bus-refresh 接口暴露出来，不然无法访问，添加配置
```
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```
三、service-client 端配置
1.添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```
2.将读取配置的位置添加注解
```
@Data
@Component
@ConfigurationProperties("config") //读取配置的上级名称
@RefreshScope //添加注解刷新配置
public class ConfigBean {
    private Integer commentlength;
}
//github 上的配置
env: dev
spring:
  application:
    name: eureka-client
server:
  port: 8763
config:
  commentlength: 400
```
> 当上边的内容配置完成后启动服务，提交git配置文件内容，调用bus-refresh 接口即可动态刷新配置。
