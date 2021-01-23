## Docker部署Sentinel
#### 测试服务 单机
    docker run --name sentinel -d \ -p 8858:8858 \ bladex/sentinel-dashboard
#### 高可用集群部署 
    =================================