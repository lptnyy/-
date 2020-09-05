# DOCKER搭建SRS-RTMP直播服务集群  
## 编译好SRS镜像IMAGE
    registry.ap-southeast-1.aliyuncs.com/leiyu/rtmp-srs:1.0  
## 一 源站服务配置文件  
    vi service.conf
    以下配置文件内容
    listen              19350;
    max_connections     1000;
    daemon              off;
    srs_log_tank        console;
    pid                 ./objs/server.pid;
    http_api {
        enabled         on;
        listen          9090;
    }
    vhost __defaultVhost__ {
        cluster {
            mode            local;
            origin_cluster  on;
            coworkers       127.0.0.1:9091;
        }
    }
## 二 边缘服务配置文件  
    vi edge.conf
    以下配置文件内容
    listen              1935;
    max_connections     1000;
    pid                 objs/edge.pid;
    daemon              off;
    srs_log_tank        console;
    vhost __defaultVhost__ {
        cluster {
           mode            remote;
           origin          source1:19351;
        }
    }