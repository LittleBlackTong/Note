# 网关的意义和网关的要素
------------------------

### 一、服务网关要素

- 稳定性，高可用
- 安全性
- 性能
- 并发性

### 二、常用网关方案

- Nginx + Lua 优势（性能极高、事件驱动涉及、扩展性好、多个功能、多个层次）
- Kong 本身基于 Nginx + Lua 商业软件花钱
- Tyk 全 restful 
- Zull 基于 JVM 路由负载均衡器 （认证、兼容、安全、限流、负载均衡等等....）一代性能有问题，二代有提升

### 三、Zuul 的特点

- 路由+过滤器 = Zuul
- 核心是一系列过滤器

### 四、Zuul 的四种过滤器API

- 前置（pre）
- 后置（post）
- 路由（route）
- 错误（error）

### 五、Zuul 声明周期
如图：
![Zull 生命周期](https://github.com/LittleBlackMann/Note/blob/master/Image/zuul-core.png?raw=true "Zull 生命周期")
