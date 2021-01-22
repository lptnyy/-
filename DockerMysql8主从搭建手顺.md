# Docker 安装mysql8主从 用于生产环境
## 虚拟机/云服务器/物理机
    mysql1 192.168.10.54 centos7
    mysql2 192.168.10.55 centos7
## 安装mysql（两台服务器）
    docker pull mysql  ## 下载最新mysql镜像 默认为mysql8.0
## 编写mysql配置文件
     