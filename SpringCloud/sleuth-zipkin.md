# 服务链路追踪 sleuth - zipkin

1. 添加依赖

```
1. 直接添加如下依赖,也可以分别添加
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>

2. 分别添加依赖

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```


2. docker 方式下载 zipkin 服务

```
docker run -d -p 9411:9411 openzipkin/zipkin  9411 是 zipkin 官方端口
```

3. 项目中添加日志的发送地址


```
spring:
	zipkin:
		base-url: http://127.0.0.1:9411/  # 链路日志发送地址
	sleuth:
		sampler:
			rate: 10 # 日志发送比率 低版本发发送率为 0.1 默认为百分之10 我用的这个版本 Hoxton.SR4 参数为 int 型 默认为 也是百分之10 
```

4. 查看链路

> 打开zipkin的控制面板 http://localhost:9411/zipkin/ 可以查看到服务与服务之间的调用情况
