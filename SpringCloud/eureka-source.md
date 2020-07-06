# EurekaClient 源码分析

### EurekaClient 的实现关系

1. 通过注解 @EnableDiscoveryClient 进入到类内部

```

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({EnableDiscoveryClientImportSelector.class})
public @interface EnableDiscoveryClient {
    boolean autoRegister() default true;
}



// 同级目录可以看到 discoveryClient

package org.springframework.cloud.client.discovery;

import java.util.List;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.core.Ordered;

public interface DiscoveryClient extends Ordered {
    int DEFAULT_ORDER = 0;

    String description();

    List<ServiceInstance> getInstances(String serviceId);

    List<String> getServices();

    default int getOrder() {
        return 0;
    }
}

```

> discoverClient 该接口为 springCloud  提供的接口 定义发现服务的抽象方法.通过搜索可以发现,EurekaDiscouveryClient 是对该接口的实现

```
import com.netflix.discovery.EurekaClient;

public class EurekaDiscoveryClient implements DiscoveryClient {
    public static final String DESCRIPTION = "Spring Cloud Eureka Discovery Client";
    private final EurekaClient eurekaClient;
    private final EurekaClientConfig clientConfig;

}
```

> 实现类依赖了 netflix.discovery.EurekaClient  而 EurekaClient 继承了 LookupService 类,而真正的实现类是 erureka 包下的 DiscveryClient


### EurekaClient 负责的任务

* 向 Eureka Server 注册服务示例
* 向 Eureka Servier 服务租约
* 当服务关闭期间, 向Eureka Server 取消租约
* 查询Eureka Server 中的服务示例列表

> Eureka Client 还需要配置一个Eureka Server 的 URL 列表

### EurekaClent 配置的服务列列表如何配置的

* 配置的名称 : eureka.client.serviceUrl.defaultZone

> 通过 serviceUrl 我们可以在 EurekaDiiscoveryClient 中发现 getServiceUrlsFromConfig 方法 但是 该方法是被 @Deprecated 声明的 说明该方法已经被弃用了
> 看源码 发现 该方法被连接 @link 到 EndpointUriles ,所以我们到 EndpointUrils中去查看

```
 public static List<String> getServiceUrlsFromConfig(EurekaClientConfig clientConfig, String instanceZone, boolean preferSameZone) {
        List<String> orderedUrls = new ArrayList<String>();
        String region = getRegion(clientConfig);
        String[] availZones = clientConfig.getAvailabilityZones(clientConfig.getRegion());
        if (availZones == null || availZones.length == 0) {
            availZones = new String[1];
            availZones[0] = DEFAULT_ZONE;
        }
        logger.debug("The availability zone for the given region {} are {}", region, availZones);
        int myZoneOffset = getZoneOffset(instanceZone, preferSameZone, availZones);

        List<String> serviceUrls = clientConfig.getEurekaServerServiceUrls(availZones[myZoneOffset]);
        if (serviceUrls != null) {
            orderedUrls.addAll(serviceUrls);
        }
        int currentOffset = myZoneOffset == (availZones.length - 1) ? 0 : (myZoneOffset + 1);
        while (currentOffset != myZoneOffset) {
            serviceUrls = clientConfig.getEurekaServerServiceUrls(availZones[currentOffset]);
            if (serviceUrls != null) {
                orderedUrls.addAll(serviceUrls);
            }
            if (currentOffset == (availZones.length - 1)) {
                currentOffset = 0;
            } else {
                currentOffset++;
            }
        }

        if (orderedUrls.size() < 1) {
            throw new IllegalArgumentException("DiscoveryClient: invalid serviceUrl specified!");
        }
        return orderedUrls;
    }


    public static String getRegion(EurekaClientConfig clientConfig) {
        String region = clientConfig.getRegion();
        if (region == null) {
            region = DEFAULT_REGION;
        }
        region = region.trim().toLowerCase();
        return region;
    }
	
	
	public String[] getAvailabilityZones(String region) {
        String value = (String)this.availabilityZones.get(region);
        if (value == null) {
            value = "defaultZone";
        }

        return value.split(",");
    }
```

> 可以看到 该方法一次加载了 getRegion()  和 getAvailabilityZones() 来获取配置,也就是 说明 一个微服务只对应了一个 Region 和 多个 Zones 他们之间是 一对多的关系 (这个配置是在区分物理机位置时来进行配置,当发生服务宕机时可以根据 region 来快速进行定位)
> eurekaClient 默认加载 defaultZone 获取 AvaliableilityZones() 时  可以看到 他是 ',' 进行区分的,defaultZone 可以加载多个
> 加载了 Region 和 Zone 之后 开始真正的加载 EurekaServer 的地址 

```
int myZoneOffset = getZoneOffset(instanceZone, preferSameZone, availZones);

List<String> serviceUrls = clientConfig.getEurekaServerServiceUrls(availZones[myZoneOffset]);
if (serviceUrls != null) {
	orderedUrls.addAll(serviceUrls);
}
```

> 通过上边的 getEurekaServerServiceUrls () 可以看一下具体的实现

```
public List<String> getEurekaServerServiceUrls(String myZone) {
        String serviceUrls = (String)this.serviceUrl.get(myZone);
        if (serviceUrls == null || serviceUrls.isEmpty()) {
            serviceUrls = (String)this.serviceUrl.get("defaultZone");
        }

        if (!StringUtils.isEmpty(serviceUrls)) {
            String[] serviceUrlsSplit = StringUtils.commaDelimitedListToStringArray(serviceUrls);
            List<String> eurekaServiceUrls = new ArrayList(serviceUrlsSplit.length);
            String[] var5 = serviceUrlsSplit;
            int var6 = serviceUrlsSplit.length;

            for(int var7 = 0; var7 < var6; ++var7) {
                String eurekaServiceUrl = var5[var7];
                if (!this.endsWithSlash(eurekaServiceUrl)) {
                    eurekaServiceUrl = eurekaServiceUrl + "/";
                }

                eurekaServiceUrls.add(eurekaServiceUrl.trim());
            }

            return eurekaServiceUrls;
        } else {
            return new ArrayList();
        }
    }
```
> 通过 Ribbon 来获取服务调用时 先访问 一个 Zone 中的服务端实例, 当这个实例为 null 时 在去访问其他的Zone 中的实例  Eureka 在进行高可用时 进行的多节点配置,就是这个时候用的

### 服务注册 

1. 通过 DiscoverClient 的构造方法 可以找到该方法 initScheduledTasks();

```
private void initScheduledTasks() {
        int renewalIntervalInSecs;
        int expBackOffBound;
        if (this.clientConfig.shouldFetchRegistry()) {
            renewalIntervalInSecs = this.clientConfig.getRegistryFetchIntervalSeconds();
            expBackOffBound = this.clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
            this.cacheRefreshTask = new TimedSupervisorTask("cacheRefresh", this.scheduler, this.cacheRefreshExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.CacheRefreshThread());
            this.scheduler.schedule(this.cacheRefreshTask, (long)renewalIntervalInSecs, TimeUnit.SECONDS);
        }

        if (this.clientConfig.shouldRegisterWithEureka()) {
            renewalIntervalInSecs = this.instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
            expBackOffBound = this.clientConfig.getHeartbeatExecutorExponentialBackOffBound();
            logger.info("Starting heartbeat executor: renew interval is: {}", renewalIntervalInSecs);
            this.heartbeatTask = new TimedSupervisorTask("heartbeat", this.scheduler, this.heartbeatExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.HeartbeatThread());
            this.scheduler.schedule(this.heartbeatTask, (long)renewalIntervalInSecs, TimeUnit.SECONDS);
            this.instanceInfoReplicator = new InstanceInfoReplicator(this, this.instanceInfo, this.clientConfig.getInstanceInfoReplicationIntervalSeconds(), 2);
            this.statusChangeListener = new StatusChangeListener() {
                public String getId() {
                    return "statusChangeListener";
                }

                public void notify(StatusChangeEvent statusChangeEvent) {
                    if (InstanceStatus.DOWN != statusChangeEvent.getStatus() && InstanceStatus.DOWN != statusChangeEvent.getPreviousStatus()) {
                        DiscoveryClient.logger.info("Saw local status change event {}", statusChangeEvent);
                    } else {
                        DiscoveryClient.logger.warn("Saw local status change event {}", statusChangeEvent);
                    }

                    DiscoveryClient.this.instanceInfoReplicator.onDemandUpdate();
                }
            };
            if (this.clientConfig.shouldOnDemandUpdateStatusChange()) {
                this.applicationInfoManager.registerStatusChangeListener(this.statusChangeListener);
            }

            this.instanceInfoReplicator.start(this.clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
        } else {
            logger.info("Not registering with Eureka server per configuration");
        }

    }
```

> 该方法中提供了 `服务注册` `服务续约` `服务获取` 的具体内容
> this.heartbeatTask = new TimedSupervisorTask("heartbeat", this.scheduler, this.heartbeatExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.HeartbeatThread());
> this.instanceInfoReplicator = new InstanceInfoReplicator(this, this.instanceInfo, this.clientConfig.getInstanceInfoReplicationIntervalSeconds(), 2);

>  heartbeatTask 开启了心跳进行服务续约  instanceInfoReplicator 进行服务注册

> new InstanceInfoReplicator() 方法我们可以看到 该类实现了 Runable 接口 通过该类启动了一个定时任务,在 该 run 方法中 .register() 方法来真正的执行服务注册的

```
public void run() {
        boolean var6 = false;

        ScheduledFuture next;
        label53: {
            try {
                var6 = true;
                this.discoveryClient.refreshInstanceInfo();
                Long dirtyTimestamp = this.instanceInfo.isDirtyWithTime();
                if (dirtyTimestamp != null) {
                    this.discoveryClient.register();
                    this.instanceInfo.unsetIsDirty(dirtyTimestamp);
                    var6 = false;
                } else {
                    var6 = false;
                }
                break label53;
            } catch (Throwable var7) {
                logger.warn("There was a problem with the instance info replicator", var7);
                var6 = false;
            } finally {
                if (var6) {
                    ScheduledFuture next = this.scheduler.schedule(this, (long)this.replicationIntervalSeconds, TimeUnit.SECONDS);
                    this.scheduledPeriodicRef.set(next);
                }
            }

            next = this.scheduler.schedule(this, (long)this.replicationIntervalSeconds, TimeUnit.SECONDS);
            this.scheduledPeriodicRef.set(next);
            return;
        }

        next = this.scheduler.schedule(this, (long)this.replicationIntervalSeconds, TimeUnit.SECONDS);
        this.scheduledPeriodicRef.set(next);
    }
```

```

boolean renew() {
        try {
            EurekaHttpResponse<InstanceInfo> httpResponse = this.eurekaTransport.registrationClient.sendHeartBeat(this.instanceInfo.getAppName(), this.instanceInfo.getId(), this.instanceInfo, (InstanceStatus)null);
            logger.debug("DiscoveryClient_{} - Heartbeat status: {}", this.appPathIdentifier, httpResponse.getStatusCode());
            if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
                this.REREGISTER_COUNTER.increment();
                logger.info("DiscoveryClient_{} - Re-registering apps/{}", this.appPathIdentifier, this.instanceInfo.getAppName());
                long timestamp = this.instanceInfo.setIsDirtyWithTime();
                boolean success = this.register();
                if (success) {
                    this.instanceInfo.unsetIsDirty(timestamp);
                }

                return success;
            } else {
                return httpResponse.getStatusCode() == Status.OK.getStatusCode();
            }
        } catch (Throwable var5) {
            logger.error("DiscoveryClient_{} - was unable to send heartbeat!", this.appPathIdentifier, var5);
            return false;
        }
    }

boolean renew() {
        try {
            EurekaHttpResponse<InstanceInfo> httpResponse = this.eurekaTransport.registrationClient.sendHeartBeat(this.instanceInfo.getAppName(), this.instanceInfo.getId(), this.instanceInfo, (InstanceStatus)null);
            logger.debug("DiscoveryClient_{} - Heartbeat status: {}", this.appPathIdentifier, httpResponse.getStatusCode());
            if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
                this.REREGISTER_COUNTER.increment();
                logger.info("DiscoveryClient_{} - Re-registering apps/{}", this.appPathIdentifier, this.instanceInfo.getAppName());
                long timestamp = this.instanceInfo.setIsDirtyWithTime();
                boolean success = this.register();
                if (success) {
                    this.instanceInfo.unsetIsDirty(timestamp);
                }

                return success;
            } else {
                return httpResponse.getStatusCode() == Status.OK.getStatusCode();
            }
        } catch (Throwable var5) {
            logger.error("DiscoveryClient_{} - was unable to send heartbeat!", this.appPathIdentifier, var5);
            return false;
        }
    }
```

> 进入到 register 方法中 我们可以看到 该方法 通过发送 http 请求来进行服务注册 其中 注册信息为 InstanceInfo (保存服务实例的相关信息)
> 同理 进入到 renew 方法中 this.instanceInfo.getAppName(), this.instanceInfo.getId(), this.instanceInfo, (InstanceStatus)null  将这些数据定时发送给 server 端进行 服务续约



### 服务获取

> 在上边的源码中 我们可以看到 this.cacheRefreshTask = new TimedSupervisorTask("cacheRefresh", this.scheduler, this.cacheRefreshExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.CacheRefreshThread());

> 服务获取和服务续约 服务注册是分开放在不同的 if 判断中的 (可以理解 注册之后 需要对服务进行续约)

> 服务获取和服务续约查看逻辑类似,当时比较复杂
```
TO-DO
```

