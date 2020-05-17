# 工作日常命令

### 连接 qa 环境服务器
> ssh root@106.75.77.69 -p 16932

### 连接 qa mysql 服务
> kubectl exec -it `kubectl get pod -o wide | grep mysql | awk '{print $1}'` bash
