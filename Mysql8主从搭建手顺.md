# 安装mysql8主从
## 虚拟机/云服务器/物理机
    mysql1 192.168.10.54 centos7
    mysql2 192.168.10.55 centos7
## 安装mysql（两台服务器）
    ## 下载mysql 选择linux系统
        https://dev.mysql.com/downloads/mysql/  
        mysql-8.0.22-1.el8.x86_64.rpm-bundle.tar
    ## 解压缩
        tar -xvf mysql-8.0.22-1.el8.x86_64.rpm-bundle.tar
    ## 删除原有的mariadb
       rpm -qa|grep mariadb  查看是否安装mariadb
       rpm -e --nodeps mariadb-libs
    ## 安装MySQL
       yum install openssl-devel.x86_64 openssl.x86_64 -y
       rpm -ivh mysql-community-common-8.0.22-1.el8.x86_64.rpm
    