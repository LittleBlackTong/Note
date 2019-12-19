# ConfigClient 实现

-----------------------

1. 修改application.yml 更改为：bootstrap.yml
2. 编写配置文件
```
spring:
  application:
    name: eureka-client //添加应用名称 要根据应用名称寻找配置文件
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG-SERVER //configserver 的配置中心服务名称
      profile: dev //配置中心的开发环境
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka/  //注册中心的默认配置要放在启动类中不然会报错
```
