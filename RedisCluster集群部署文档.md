# redis6 集群搭建手顺
## 虚拟机/云服务器/物理服务器 推荐3台服务器 搭建 3主 3从
   192.168.10.53 redis_server
   
## 安装redis
    ## 下载 
        wget http://download.redis.io/releases/redis-6.0.6.tar.gz
        tar zxvf  redis-6.0.6.tar.gz  # 解压缩
    ## 注意事项 如果是centos 7 编译前运行以下命令升级部分组件
        sudo yum install centos-release-scl
        sudo yum install devtoolset-7-gcc*
        scl enable devtoolset-7 bash
    ## 编译
        cd redis目录
        执行命令 make  # 如果系统此命令不存在 安装   yum -y install gcc automake autoconf libtool make   yum -y install gcc gcc-c++
## 创建文件夹，修改配置文件
    ## 创建六个文件文件夹存放配置文件
        mkdir redis1
        mkdir redis2
        mkdir redis3
        mkdir redis4
        mkdir redis5
        mkdir redis6
    ## 修改配置文件  
        编辑程序根目录下的redis.conf
        vi redis.conf
        port 7000
        bind 0.0.0.0
        cluster-enabled yes
        cluster-config-file nodes.conf
        cluster-node-timeout 5000
        appendonly yes
    ## 修改完配置文件 copy 配置文件到不同的目录中
       cp redis.conf /root/redis1
       cp redis.conf /root/redis2
       cp redis.conf /root/redis3
       cp redis.conf /root/redis4
       cp redis.conf /root/redis5
       cp redis.conf /root/redis6
    ## 修改配置文件端口号
       redis1 端口7001   port
       cluster-config-file nodes1.conf 
       redis2 端口7002   port
       cluster-config-file nodes2.conf
       redis3 端口7003   port
       cluster-config-file nodes3.conf
       redis4 端口7004   port
       cluster-config-file nodes4.conf
       redis5 端口7005   port
       cluster-config-file nodes5.con       redis6 端口7006   port
       cluster-config-file nodes6.conf
       
## 运行集群
       ## 运行各个redis服务  cd /解压目录/src
       ./redis-server /root/redis1/redis.conf
       ./redis-server /root/redis2/redis.conf
       ./redis-server /root/redis3/redis.conf
       ./redis-server /root/redis4/redis.conf
       ./redis-server /root/redis5/redis.conf
       ./redis-server /root/redis6/redis.conf
       ## 运行集群命令
            ./redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
       ## 运行结果 安装成功已经集群完毕
        >>> Performing hash slots allocation on 6 nodes...
        Master[0] -> Slots 0 - 5460
        Master[1] -> Slots 5461 - 10922
        Master[2] -> Slots 10923 - 16383
        Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
        Adding replica 127.0.0.1:7006 to 127.0.0.1:7002
        Adding replica 127.0.0.1:7004 to 127.0.0.1:7003
        >>> Trying to optimize slaves allocation for anti-affinity
        [WARNING] Some slaves are in the same host as their master
        M: 2b8581ae41d4dffc9c86b362fdcde6b6cec7ace3 127.0.0.1:7001
           slots:[0-5460] (5461 slots) master
        M: 0eb69351b8a6f89d75fc11f74fc1f1eea3f9a48a 127.0.0.1:7002
           slots:[5461-10922] (5462 slots) master
        M: 57cd52886f6dc0813e4d53e90f781dee1887a71f 127.0.0.1:7003
           slots:[10923-16383] (5461 slots) master
        S: fe5a1caf45de33c18506ab15327eb4a0650d96ff 127.0.0.1:7004
           replicates 0eb69351b8a6f89d75fc11f74fc1f1eea3f9a48a
        S: f25d973731386e302c374100e35d8f5f5e89ebf9 127.0.0.1:7005
           replicates 57cd52886f6dc0813e4d53e90f781dee1887a71f
        S: eba42fb709cc4df212808bf3bcf0d60d2e9fedde 127.0.0.1:7006
           replicates 2b8581ae41d4dffc9c86b362fdcde6b6cec7ace3
        Can I set the above configuration? (type 'yes' to accept): yes
        >>> Nodes configuration updated
        >>> Assign a different config epoch to each node
        >>> Sending CLUSTER MEET messages to join the cluster
        Waiting for the cluster to join
        .
        >>> Performing Cluster Check (using node 127.0.0.1:7001)
        M: 2b8581ae41d4dffc9c86b362fdcde6b6cec7ace3 127.0.0.1:7001
           slots:[0-5460] (5461 slots) master
           1 additional replica(s)
        S: f25d973731386e302c374100e35d8f5f5e89ebf9 127.0.0.1:7005
           slots: (0 slots) slave
           replicates 57cd52886f6dc0813e4d53e90f781dee1887a71f
        S: eba42fb709cc4df212808bf3bcf0d60d2e9fedde 127.0.0.1:7006
           slots: (0 slots) slave
           replicates 2b8581ae41d4dffc9c86b362fdcde6b6cec7ace3
        M: 57cd52886f6dc0813e4d53e90f781dee1887a71f 127.0.0.1:7003
           slots:[10923-16383] (5461 slots) master
           1 additional replica(s)
        M: 0eb69351b8a6f89d75fc11f74fc1f1eea3f9a48a 127.0.0.1:7002
           slots:[5461-10922] (5462 slots) master
           1 additional replica(s)
        S: fe5a1caf45de33c18506ab15327eb4a0650d96ff 127.0.0.1:7004
           slots: (0 slots) slave
           replicates 0eb69351b8a6f89d75fc11f74fc1f1eea3f9a48a
        [OK] All nodes agree about slots configuration.
        >>> Check for open slots...
        >>> Check slots coverage...
        [OK] All 16384 slots covered.
