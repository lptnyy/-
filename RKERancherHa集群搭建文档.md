# Rancher Ha 集群搭建手顺
## 虚拟机/云服务器/物理服务器
    rancher_server1 192.168.10.50 (centos 7)
    rancher_server2 192.168.10.51 (centos 7)
    work_server1 192.168.10.52 (centos 7)
## ssh 工具
    xshell 
## 更新系统(所有服务器)
    xshell登录所有服务器,工具栏勾选把所有命令输入到所有会话中，执行下面的命令。  
    yum update -y
## 同步服务器时间（所有服务器）
    xshell登录所有服务器,工具栏勾选把所有命令输入到所有会话中，执行下面的命令。  
    （1）更改时区
        cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
     (2) 安装nepdate
        yum install ntpdate -y
     (3) 同步时间
        ntpdate ntp1.aliyun.com
     (4) 保存时间设置
        clock -w
## 关闭swap分区（所有服务器）
    echo vm.swappiness=0 >> /etc/sysctl.con
    reboot 重启服务器
    通过 free -g 查看swap分区是否全部为0
    