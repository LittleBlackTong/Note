# ConfigServer 实现方式

----------------------

### 1. 创建项目时选择eureka Discovery 服务发现
### 2. 启动类添加注解

```
@EnableConfigServer
```

### 3. 编写配置文件

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/LittleBlackMann/Config-Server.git  //远端仓库的地址
          username: //远端仓库的用户名
          password: // 远端仓库的密码
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/ //注册中心地址
```
### 4. github 上传配置文件

如图：
![文件截图](https://github.com/LittleBlackMann/Note/blob/master/Image/config-server1.png?raw=true "配置中心文件")

> 文件的命名格式为：applicationname-environment 方式
> config-server 就会根据各个环境文件名称去寻找文件并且下载到指定的目录。
