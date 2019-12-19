# 一、RestTemplate
### 方式1.直接写死 url 通过 resttemplate 实例获取
```
RestTemplate restTemplate = new RestTemplate();
String response = restTemplate.getForObject("http://localhost:8763/msg", String.class);
```

### 方式2. 采用字符串拼接的方式通过 loadbalance 获取 `host` 以及 `port`

```
ServiceInstance serviceInstance = loadBalancerClient.choose("EUREKA_CLIENT");
String url = String.format("http://%s:%s/msg", serviceInstance.getHost(), serviceInstance.getPort());
String response = restTemplate.getForObject(url, String.class);
```
### 方式3.利用 @loadBalance 注解进行注入
```
1.restTemplate 自动实现类

@Component
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}


2.自动注入
//方式三 利用 @LoadBalanced 自动注入
String response = restTemplate.getForObject("http://eureka-client/msg", String.class);
```
> 注意：消费者需要添加 pom 依赖，不然会报错无法找到实例

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
# 二、客户端负载均衡
### 1.Ribbon
>核心：1.服务发现（发现服务）2.服务选择规则（选择有效服务）3.服务监听（检测失效服务）
>主要组件： 1.serverlist 2.irule 3.serverlistfilter
