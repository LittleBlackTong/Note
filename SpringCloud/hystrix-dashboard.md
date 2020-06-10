# 整合 hystrix dashboard 观看服务熔断情况

1. 添加依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
	<version>2.2.2.RELEASE</version>
</dependency>
```
2. 在启动类上添加注解

```
@EnableHystrixDashboard

```
3. 打开控制面板

`http://127.0.0.1:5620/hystrix`

4. 输入监听地址

`http://127.0.0.1:5620/hystrix.stream`

> 这个地方大概率提示 Unable to connect to Command Metric Stream  这是需要检查 1.启动类注解是否添加 2.监听的地址是否正确(springCloud 有一些版本 地址变更为:http://127.0.0.1:5620/actuator/hystrix.stream)
> 如果地址正确打开监听时出现 404 错误 可以添加如下配置(也就是将监听地址映射成: /hystrix.stream)

```
    @Bean
    public ServletRegistrationBean<HystrixMetricsStreamServlet> getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean;
        registrationBean = new ServletRegistrationBean<>(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
```
