# 一·注册中心
### 1.服务的发现方式
> (1)客户端发现 eureka
  (2)服务端发现 nginx zookeeper kubernetes

### 2.服务的调用方式
> (1)REST or RPC eureka 使用的是 REST 方式

### 3.服务端代码实现

（1）开启服务端注解
```
@SpringBootApplication  
@EnableEurekaServer //开启 eureka 服务端注解
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```
 （2）编写配置文件
```
spring:
  application:
    name: eureka  //服务名称
server:
  port: 8761 //服务端口
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/ //注册地址
    fetch-registry: false 
    register-with-eureka: true //是否注册到 eureka
  server:
    enable-self-preservation: false //是否允许自己注册自己
```
### 4.客户端代码实现
（1）开启注解
```
@SpringBootApplication
@EnableDiscoveryClient //开启注解服务发现
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```
（2）编写配置文件
```
spring:
  application:
    name: eureka_client //服务名称
server:
  port: 8763 //服务端口
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/,http://127.0.0.1:8762/eureka/ //注册到那个注册中心
```
###5.eureka 的高可用
（1）配置文件 eureka server 互相注册即可
```
1.server1

spring:
  application:
    name: eureka
server:
  port: 8761
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8762/eureka/
    fetch-registry: false
    register-with-eureka: true
  server:
    enable-self-preservation: false
    
2.server2

spring:
  application:
    name: eureka
server:
  port: 8762
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/
    fetch-registry: false
    register-with-eureka: true
  server:
    enable-self-preservation: false
```
