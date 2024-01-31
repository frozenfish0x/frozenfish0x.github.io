---
title: kvm虚拟机安装wrt
date: 2024-01-08 16:29:59
tags:
---
# 最近在折腾服务部署公网的事情

```
本来想着360t7刷成wrt以为能在线涮实际上是要烧录
他自带的防火墙挡我的服务却没有关闭的地方。这个就很难受
因为一时半会没有办法整一台拥有wrt的硬路由
又听虚拟机kvm和vbox。选择kvm虚拟机
之前用docker结果夺舍宿主机，。这个我不太了解dockers技术等以后会研究一下他的虚拟化技术！！

服务器环境是arch。kvm。docker，ddns，反向代理还有https以后会考虑搞！
```

搬运自：[archlinux 安装KVM,QEMU - 掰断曲别针 - 博客园 (cnblogs.com)](https://www.cnblogs.com/suxiuf/p/17586593.html)

KVM是Linux世界中最常用的虚拟化软件之一。事实上，大多数云提供商都使用KVM作为他们的虚拟机管理程序。包括OpenStack在内的大型项目都使用KVM作为默认虚拟化工具。

以下是如何在Arch Linux和Manjaro上安装KVM、QEMU和Virt Manager的完整指南。
<!--more-->
# 安装KVM包

第一步是安装运行KVM所需的所有软件包：

```shell
sudo pacman -Syy
sudo pacman -S archlinux-keyring
sudo pacman -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat dmidecode
```

同时安装ebtbles和iptables软件包。

```shell
sudo pacman -S ebtables iptables
```

# 安装其他工具

libguestfs是一组用于访问和修改虚拟机（VM）磁盘映像的工具。作用如下：

- 查看和编辑来宾中的文件
- 编写对虚拟机的更改脚本
- 监视磁盘使用/可用统计信息
- 创建来宾
- P2V （物理机到虚拟机的迁移）
- V2V （虚拟机到虚拟机的迁移）
- 执行备份及其他

```shell
sudo pacman -S libguestfs
```

# 启动KVM libvirt服务

- 启用服务并设置开机自启动

```shell
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
```

- 查看运行状态

```shell
systemctl status libvirtd.service
```

# 配置普通用户可以使用KVM

- 打开/etc/libvirt/libvirtd.conf文件进行编辑。

```shell
sudo pacman -S vim
sudo vim /etc/libvirt/libvirtd.conf
```

- 将UNIX域套接字组所有权设置为libvirt（第85行左右）

```shell
unix_sock_group = "libvirt"
```

- 为R/W套接字设置UNIX套接字权限（第102行附近）

```shell
unix_sock_rw_perms = "0770"
```

- 将当前用户帐户添加到libvirt组

```shell
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt
```

- 重新启动libvirt守护进程。

```shell
sudo systemctl restart libvirtd.service
```

# 启用嵌套虚拟化（可选）

- 嵌套虚拟化就是在虚拟机中运行虚拟机。
  如图所示，通过启用内核模块为kvm_intel / kvm_amd启用嵌套虚拟化。

一般不会这样搞。

```shell
### Intel Processor ###
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1

### AMD Processor ###
sudo modprobe -r kvm_amd
sudo modprobe kvm_amd nested=1
```

- 要使此配置持久化，请运行：

```shell
echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```

- 确认“嵌套虚拟化”设置为“yes”：

```shell
## Intel Processor ###
$ systool -m kvm_intel -v | grep nested
    nested              = "Y"
    nested_early_check  = "N"
$ cat /sys/module/kvm_intel/parameters/nested 
Y

### AMD Processor ###
$ systool -m kvm_amd -v | grep nested
    nested              = "Y"
    nested_early_check  = "N"
$ cat /sys/module/kvm_amd/parameters/nested 
Y
```





# kvm介绍

转载：[KVM的基本使用 - 匿名者nwnu - 博客园 (cnblogs.com)](https://www.cnblogs.com/nwnusun/p/16574336.html)

虚拟化是云计算的基础。简单的说，虚拟化使得在一台物理的服务器上可以跑多台虚拟机，虚拟机共享物理机的 CPU、内存、IO 硬件资源，但逻辑上虚拟机之间是相互隔离的。

物理机我们一般称为宿主机（Host），宿主机上面的虚拟机称为客户机（Guest）。

那么 Host 是如何将自己的硬件资源虚拟化，并提供给 Guest 使用的呢？ 这个主要是通过一个叫做 Hypervisor 的程序实现的。

根据 Hypervisor 的实现方式和所处的位置，虚拟化又分为两种：

- 全虚拟化- 半虚拟化
  **全虚拟化：** Hypervisor 直接安装在物理机上，多个虚拟机在 Hypervisor 上运行。Hypervisor 实现方式一般是一个特殊定制的 Linux 系统。Xen 和 VMWare 的 ESXi 都属于这个类型

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314132441934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314132441934.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70) **半虚拟化：**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314132539401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314132539401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**理论上讲：**

全虚拟化一般对硬件虚拟化功能进行了特别优化，性能上比半虚拟化要高； 半虚拟化因为基于普通的操作系统，会比较灵活，比如支持虚拟机嵌套。**嵌套意味着可以在KVM虚拟机中再运行KVM。**

***2\***|***0\*****2. kvm介绍**

kVM 全称是 Kernel-Based Virtual Machine。也就是说 KVM 是基于 Linux 内核实现的。 KVM有一个内核模块叫 kvm.ko，只用于管理虚拟 CPU 和内存。

那 IO 的虚拟化，比如存储和网络设备则是由 Linux 内核与Qemu来实现。

作为一个 Hypervisor，KVM 本身只关注虚拟机调度和内存管理这两个方面。IO 外设的任务交给 Linux 内核和 Qemu。

大家在网上看 KVM 相关文章的时候肯定经常会看到 Libvirt 这个东西。

Libvirt 就是 KVM 的管理工具。

其实，Libvirt 除了能管理 KVM 这种 Hypervisor，还能管理 Xen，VirtualBox 等。

Libvirt 包含 3 个东西：后台 daemon 程序 libvirtd、API 库和命令行工具 virsh

- libvirtd是服务程序，接收和处理 API 请求；
- API 库使得其他人可以开发基于 Libvirt 的高级工具，比如 virt-manager，这是个图形化的 KVM 管理工具；
- virsh 是我们经常要用的 KVM 命令行工具

***3\***|***0\*****3. kvm部署**

**环境说明：**

IP：192.168.157.99

***3\***|***1\*****3.1 kvm安装**

部署前请确保你的CPU虚拟化功能已开启。分为两种情况：

- 虚拟机要关机设置CPU虚拟化- 物理机要在BIOS里开启CPU虚拟化
  **//关闭防火墙与selinux**



```
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -ri 's/^(SELINUX=).*/\1disabled/g' /etc/selinux/config
//这一步十分重要！！！
```

**//配置网络源**



```
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum makecache
yum -y install epel-release vim wget net-tools unzip zip gcc gcc-c++
```

**//验证CPU是否支持KVM；如果结果中有vmx（Intel）或svm(AMD)字样，就说明CPU的支持的**



```
egrep -o 'vmx|svm' /proc/cpuinfo
```

**//安装KVM依赖包及管理工具**



```
# kvm属于内核态，不需要安装。但是需要一些管理工具包
yum install -y qemu-kvm libvirt libvirt-python libguestfs-tools virt-install virt-manager python-virtinst libvirt-client virt-viewer qemu-kvm-tool
```

**//因为虚拟机中网络，我们一般都是和公司的其他服务器是同一个网段，所以我们需要把 KVM服务器的网卡配置成桥接模式。这样的话KVM的虚拟机就可以通过该桥接网卡和公司内部 其他服务器处于同一网段**

**//此处我的网卡是ens32，所以用br0来桥接ens32网卡**



```
//对应修改或者添加以下内容即可
[root@mp ~]# vim /etc/sysconfig/network-scripts/ifcfg-br0 
TYPE=Bridge
DEVICE=br0
NM_CONTROLLED=no
BOOTPROTO=static
NAME=br0
ONBOOT=yes
IPADDR=192.168.157.99
NETMASK=255.255.255.0
GATEWAY=192.168.157.2
DNS1=8.8.8.8

//保留以下内容即可
[root@mp ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens32 
TYPE=Ethernet
BOOTPROTO=static
NAME=ens33
DEVICE=ens33
ONBOOT=yes
BRIDGE=br0
NM_CONTROLLED=no
[root@mp ~]# systemctl restart network
```

**//启动服务**



```
systemctl start libvirtd
systemctl enable libvirtd
```

**//验证安装结果**



```
[root@mp ~]# lsmod|grep kvm
kvm_intel             170086  0 
kvm                   566340  1 kvm_intel
irqbypass              13503  1 kvm
```

**//测试并验证安装结果**



```
[root@mp ~]# virsh -c qemu:///system list
 Id    名称                         状态
----------------------------------------------------
[root@mp ~]# virsh --version
4.5.0
[root@mp ~]# virt-install --version
1.5.0
[root@mp ~]# ln -s /usr/libexec/qemu-kvm /usr/bin/qemu-kvm
[root@mp ~]# ll /usr/bin/qemu-kvm 
lrwxrwxrwx. 1 root root 21 3月  14 14:17 /usr/bin/qemu-kvm -&  /usr/libexec/qemu-kvm
	
[root@mp ~]# lsmod |grep kvm
kvm_intel             170086  0 
kvm                   566340  1 kvm_intel
irqbypass              13503  1 kvm
```

**//查看网桥信息**



```
[root@mp ~]# brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.000c2993a66c	no		ens32
virbr0		8000.5254009df26a	yes		virbr0-nic
```

***3\***|***2\*****3.2 kvm web管理界面安装**

kvm 的 web 管理界面是由 webvirtmgr 程序提供的。

**//安装依赖包**



```
# 管理端安装
yum -y install git python-pip libvirt-python libxml2-python python-websockify supervisor gcc python-devel
# 使用pip安装Python扩展程序库
pip install numpy -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
```

**//升级pip**



```
 pip install --upgrade pip
```

**//git克隆配置并运行WebVirMgr**



```
# 创建data目录，将WebVirtMgr移动到data目录，同时创建KVM存储目录
mkdir /data/kvm -pv
# 克隆项目
cd /data
git clone git://github.com/retspen/webvirtmgr.git
cd webvirtmgr
pip install -r requirements.txt  -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
# 说明:requirements.txt主要是用于记录所有依赖包及其精确的版本号。以便新环境部署
# 初始化环境
./manage.py syncdb
## 这里需要我们输入Yes，配置管理员用户
配置信息如下
Would you like to create one now? (yes/no): yes         #是否现在创建管理员用户
Username (leave blank to use 'root'): root              #用户名称
Email address:                                          #邮箱地址 (可以不填)
Password:                                               #管理员用户密码
Password (again):                                       #重复输入密码
Superuser created successfully.                         #创建成功
# 配置Django 静态页面
./manage.py collectstatic 
输入Yes即可
# 如果还想继续添加管理员用户，可以执行下面的命令
./manage.py createsuperuser
```

//**启动WebVirMgr**



```
前台启动WebVirMgr，默认是Debug模式同时日志打印在前台
./manage.py runserver 0:8000
```

访问:[http://192.168.209.134:8000](http://192.168.209.134:8000/)

**//安装Nginx**



```
1.安装依赖
yum install -y gcc glibc gcc-c++ prce-devel openssl-devel pcre-devel
2.安装编译Nginx
cd /root/
wget http://nginx.org/download/nginx-1.16.1.tar.gz
useradd -s /sbin/nologin nginx -M 
tar xf nginx-1.16.1.tar.gz && cd nginx-1.16.1.tar.gz
./configure --prefix=/usr/local/nginx-1.16.1 --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module
make && make install
ln -s /usr/local/nginx-1.16.1 /usr/local/nginx
# 修改配置文件
cd /usr/local/nginx/conf
vim nginx.conf

worker_processes  1;
user nginx;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  kvm.i4t.com;
      
        location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $remote_addr;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 5120M;
          }
    location /static/ {
        root /data/webvirtmgr;
        expires max;
      }
    }

}
# 启动Nginx
/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx 
```

**//配置nginx**



```
1.安装依赖
yum install -y gcc glibc gcc-c++ prce-devel openssl-devel pcre-devel
2.安装编译Nginx
cd /root/
wget http://nginx.org/download/nginx-1.16.1.tar.gz
useradd -s /sbin/nologin nginx -M 
tar xf nginx-1.16.1.tar.gz && cd nginx-1.16.1.tar.gz
./configure --prefix=/usr/local/nginx-1.16.1 --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module
make && make install
ln -s /usr/local/nginx-1.16.1 /usr/local/nginx
# 修改配置文件
cd /usr/local/nginx/conf
vim nginx.conf

worker_processes  1;
user nginx;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  kvm.i4t.com;
      
        location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $remote_addr;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 5120M;
          }
    location /static/ {
        root /data/webvirtmgr;
        expires max;
      }
    }

}
# 启动Nginx
/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx 
```

**//创建supervisor配置文件**



```
WebVirtMgr默认使用supervisor进行管理(启动停止服务)

cat > /etc/supervisord.d/webvirtmgr.ini << EOF
[program:webvirtmgr]
command=/usr/bin/python /data/webvirtmgr/manage.py run_gunicorn -c /data/webvirtmgr/conf/gunicorn.conf.py
directory=/data/webvirtmgr
autostart=true
autorestart=true
logfile=/var/log/supervisor/webvirtmgr.log
log_stderr=true
user=root

[program:webvirtmgr-console]
command=/usr/bin/python /data/webvirtmgr/console/webvirtmgr-console
directory=/data/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr-console.log
redirect_stderr=true
user=root
EOF
# 启动
systemctl start supervisord 
systemctl enable supervisord
# 检查
supervisorctl status
# 重启所以（非执行）
supervisorctl restart all
```

**//重启nginx**



```
/usr/local/nginx/sbin/nginx -s reload
```

***4\***|***0\*****4. kvm web界面管理**

通过ip地址在浏览器上访问kvm，例如我这里就是：

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314192444173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314192444173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)此处的用户为：root 密码为：执行python manage syncdb时设置的超级管理员密码

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314193031866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314193031866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)此处的Label要与下面的FQDN / IP一致！

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314193543699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314193543699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)点击上方的IP地址，不是点击Host：192.168.157.99

**3.3.2 kvm存储管理**

**//创建存储**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314193925599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314193925599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)点击New Storage

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314194036367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314194036367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)****

**进入存储** [![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314194133477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314194133477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)点击default

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314194253161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314194253161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)池路径 /var/lib/libvirt/images：磁盘镜像ISO文件存储的位置

**//通过远程连接软件上传ISO镜像文件至存储目录/var/lib/libvirt/images/**



```
[root@mp ~]# cd /var/lib/libvirt/images/
[root@mp images]# ll
总用量 3963904
-rw-r--r-- 1 root root 4059037696 3月  15 03:50 rhel-server-7.4-x86_64-dvd.iso
```

**//在web界面查看ISO镜像文件是否存在** [![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314195305198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314195305198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**//创建系统安装镜像** [![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314195505238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314195505238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**//添加成功如下图** [![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314195604723.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314195604723.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**3.3.3 kvm网络管理**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314195711201.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314195711201.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)点击New Network

[![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031419593859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2019031419593859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)**3.3.4 实例管理**

**实例（虚拟机的创建）** [![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314200516947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314200516947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**//虚拟机插入光盘**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314200608305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314200608305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**//设置在web上访问虚拟机的密码**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314200729314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314200729314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**//启动虚拟机**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314200838724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314200838724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)[![在这里插入图片描述](https://img-blog.csdnimg.cn/20190314200917131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190314200917131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)

**//虚拟机安装**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031420163376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2019031420163376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)此步骤为虚拟机的安装步骤，不再阐述

***5\***|***0\*****4. 所遇问题*****5\***|***1\*****4.1 故障一**

第一次通过web访问kvm时可能会一直访问不了，一直转圈，而命令行界面一直报错(too many open files)



```
	永久生效方法：
		修改/etc/security/limits.conf，在文件底部添加：
		* soft nofile 655360
		* hard nofile 655360
		星号代表全局， soft为软件，hard为硬件，nofile为这里指可打开文件数。
	 
	另外，要使limits.conf文件配置生效，必须要确保 pam_limits.so 文件被加入到启动文件中。
	查看 /etc/pam.d/login 文件中有：
	session required /lib/security/pam_limits.so
```

***5\***|***2\*****4.2 故障二**

web界面配置完成后可能会出现以下错误界面 [![在这里插入图片描述](https://img-blog.csdnimg.cn/20190318150017680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20190318150017680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY5NTEwNA==,size_16,color_FFFFFF,t_70) 解决方法是安装novnc并通过novnc_server启动一个vnc



```
	[root@mp ~]# ll /etc/rc.local
	lrwxrwxrwx. 1 root root 13 Aug  6  2018 /etc/rc.local -&gt; rc.d/rc.local
	[root@mp ~]# ll /etc/rc.d/rc.local
	-rw-r--r-- 1 root root 513 Mar 11 22:35 /etc/rc.d/rc.local
	[root@mp ~]# chmod +x /etc/rc.d/rc.local
	[root@mp ~]# ll /etc/rc.d/rc.local
	-rwxr-xr-x 1 root root 513 Mar 11 22:35 /etc/rc.d/rc.local
	
	[root@mp ~]# vim /etc/rc.d/rc.local
	......此处省略N行
	# that this script will be executed during boot.
	
	touch /var/lock/subsys/local
	nohup novnc_server 172.16.12.128:5920 &amp;
	
	[root@mp ~]# . /etc/rc.d/rc.local
```

# 安装yay aur包管理器

```bash

git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si 
```

# 通过aur安装

```bash
yay -S webvirtmgr-git
yaourt -S webvirtmgr-git
```

# 如果连接失败

```bash
可以使用wattlk也就是steam++
命令如下
curl -sSL https://steampp.net/Install/Linux.sh | bash
然后在尝试
```

# 采用命令行自带的管理进行安装虚拟机

```
这steam++速度也不是很行
```

centos debian



# archlinux牵扯问题：

```bash
内核加载等，我不想折腾了。于是选择更换稳定的发行版。
用arch做服务器是我今年做过最愚蠢的事情之一了
```

