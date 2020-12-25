# Rancher Ha 集群搭建手顺
## 虚拟机/云服务器/物理服务器 k8s rancher etcd 正式环境部署最少3节点
    rancher_server1 192.168.10.50 (centos 7) nginx
    rancher_server2 192.168.10.51 (centos 7) k8s rancher etcd
    rancher_server3 192.168.10.52 (centos 7) k8s rancher etcd
## ssh 工具
    xshell 
## 更新系统(所有服务器)
    xshell登录所有服务器,工具栏勾选把所有命令输入到所有会话中，执行下面的命令。  
    yum update -y
## 关闭防火墙(所有服务器)
    systemctl stop firewalld
    systemctl disable firewalld
    不关闭防火墙需要防火墙对外公开 6443端口
## 关闭setlinx(所有服务器)
    sudo setenforce 0
    grep SELINUX /etc/selinux/config
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
    swapoff -a
    修改配置文件 - /etc/fstab
    删除swap相关行 /mnt/swap swap swap defaults 0 0 这一行或者注释掉这一行
    echo vm.swappiness=0 >> /etc/sysctl.conf
    reboot 重启服务器
    通过 free -g 查看swap分区是否全部为0
    如果 swap 都不为0 就设置失败了 通过查找命令 查找已经有设置 vm.swappiness=0 的配置文件 全部修改为0
    find /usr/lib/tuned -name '*.conf' -type f -exec grep "vm.swappiness" {} \+
## 文件打开数调优（所有服务器）
    echo -e  "root soft nofile 65535\nroot hard nofile 65535\n* soft nofile 65535\n* hard nofile 65535\n"     >> /etc/security/limits.conf
    sed -i 's#4096#65535#g' /etc/security/limits.d/20-nproc.conf
## 内核调优（所有服务器）
    cat >> /etc/sysctl.conf<<EOF
    net.ipv4.ip_forward=1
    net.bridge.bridge-nf-call-iptables=1
    net.bridge.bridge-nf-call-ip6tables=1
    vm.max_map_count=655360
    EOF
## 安装部分依赖（所有服务器）
    yum -y install wget ntpdate lrzsz curl yum-utils device-mapper-persistent-data lvm2 bash-completion 
## 修改机器名（所有服务器）
    hostnamectl set-hostname rancher_server1 
    hostnamectl set-hostname rancher_server2 
    hostnamectl set-hostname rancher_server3
## 添加hosts（所有服务器）
    192.168.10.50 rancher_server1
    192.168.10.51 rancher_server2
    192.168.10.52 rancher_server3
    reboot 重启服务
## 创建用户组（所有服务器）
    groupadd docker
    useradd rancher -G docker
    echo "123456" | passwd --stdin rancher
## 修改ssh conf文件（所有服务器）
    vi /etc/ssh/sshd_config 
    AllowTcpForwarding yes 删除#号
    service sshd restart 重启ssh服务
## 配置ssh免登录
    #只在rancher_server1 执行
        su - rancher
        ssh-keygen
        ssh-copy-id rancher@192.168.10.50
        ssh-copy-id rancher@192.168.10.51
        ssh-copy-id rancher@192.168.10.52
## 安装docker(所有服务器)
    # 删除 系统自带docker
        yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  container*
    curl https://releases.rancher.com/install-docker/18.09.2.sh | sh
    systemctl enable docker
    systemctl restart docker
## 配置docker 阿里云源(所有服务器)
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://jehbigkp.mirror.aliyuncs.com"]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker
## 安装RKE (rancher_server1)
    # 下载官方最新包
        wget https://github.com/rancher/rke/releases/download/v1.2.4-rc9/rke_linux-arm64
        mv rke_linux-arm64 /usr/bin/rke
        chnod 777 /usr/bin/rke
## 编写RKE配置文件安装Kubernetes(k8s)集群环境（rancher_server1）
    vi rancher-cluster.yml 
    nodes:
      - address: 192.168.10.51
        user: rancher
        role: [controlplane,worker,etcd]
      - address: 192.168.10.52
        user: rancher
        role: [controlplane,worker,etcd]
    services:
      etcd:
        snapshot: true
        creation: 6h
        retention: 24h
    # 执行命令  安装集群环境 环境创建成功之后 会生成一个配置文件 
        rke up --config rancher-cluster.yml  自动生成kube_config_rancher-cluster.yml 就是集群文件需要好好保存
    # 配置环境变量
        export KUBECONFIG=/路径/rancher-cluster.yml 
            
## 安装kubectl 查看集群环境（rancher_server1）
    # 切换到root用户  
        su root
    # 下载地址 
        https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#client-binaries 选择最新版本
    # 下载 
        kubernetes-client-linux-arm.tar.gz  tar zxvf kubernetes-client-linux-arm.tar.gz  解压
    # 将kubectl文件拷贝到 /usr/bin
        mv kubectl /usr/bin/
        su rancher
    # 执行命令查看集群是否成功
        kubectl get nodes  显示以下内容表示集群成功
        ------------------------------------------------------------------------------------------------------
        NAME            STATUS   ROLES                      AGE   VERSION
        192.168.10.51   Ready    controlplane,etcd,worker   85m   v1.19.6
        192.168.10.52   Ready    controlplane,etcd,worker   85m   v1.19.6
## 安装helm（rancher_server1）
    # 下载地址  
        https://github.com/helm/helm/releases/tag/v3.4.2 
    # 解压 
        tar zxvf helm-v3.4.2-linux-arm64.tar.gz
    # 执行 
        mv helm /usr/bin
    # 安装 Helm Chart 仓库
        helm repo add rancher-stable http://rancher-mirror.oss-cn-beijing.aliyuncs.com/server-charts/stable
## 安装证书服务cert-manager(rancher_server1) 
    # k8s 环境创建命名空间
        kubectl create namespace cattle-system
    # 安装 cert-manager
        kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager.crds.yaml
    # 创建命名空间 
        kubectl create namespace cert-manager
    # 添加 Jetstack Helm 仓库
        helm repo add jetstack https://charts.jetstack.io
    # 更新本地 Helm chart 仓库缓存
        helm repo update
    # 安装 cert-manager Helm chart
        helm install \
         cert-manager jetstack/cert-manager \
         --namespace cert-manager \
         --version v0.15.0
    # 验证是否部署成功 cert-manager
        kubectl get pods --namespace cert-manager   获取该命名空间下的所有pod 
## 安装rancher(rancher_server1)
    # 此方式部署的是rancher自动生成证书方式  自有证书查看官网介绍。 
      官方地址 https://docs.rancher.cn/docs/rancher2/installation/k8s-install/helm-rancher/_index
    # 安装命令
        helm install rancher rancher-stable/rancher \
         --namespace cattle-system \
         --set hostname=rancher.my.org
    # 等待安装结果
        kubectl -n cattle-system rollout status deploy/rancher
        Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
        deployment "rancher" successfully rolled out
    # 访问域名   安装命令当中的 --set hostname=rancher.my.org
      rancher.my.org
## 配置 nginx （搭建省略）
    # 配置nginx 配置文件 
        vi nginx.conf
        输入一下内容 
        worker_processes 4;
        worker_rlimit_nofile 40000;
        
        events {
            worker_connections 8192;
        }
        
        stream {
            upstream rancher_servers_http {
                least_conn;
                server 192.168.10.51:80 max_fails=3 fail_timeout=5s;
                server 192.168.10.52:780 max_fails=3 fail_timeout=5s;
            }
            
            server {
                listen     80;
                proxy_pass rancher_servers_http;
            }
        
            upstream rancher_servers_https {
                least_conn;
                server 192.168.10.51:443 max_fails=3 fail_timeout=5s;
                server 192.168.10.52:443 max_fails=3 fail_timeout=5s;
            }
            
            server {
                listen     443;
                proxy_pass rancher_servers_https;
            }
        }
    # 下载nginx docker镜像
## 如果在局域网中访问域名 配置hosts文件(访问rancher的电脑)
    192.168.10.51 rancher.my.org     ip 可以是nginx ip  也可以是集群当中某一台服务器的ip