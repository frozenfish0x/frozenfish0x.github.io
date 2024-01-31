---
title: 转debian
date: 2024-01-08 22:36:12
tags:
---

# 系统相关

```
fish@fishserver:~$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0 238.5G  0 disk 
├─sda1                      8:1    0   512M  0 part /boot/efi
├─sda2                      8:2    0   488M  0 part /boot
└─sda3                      8:3    0 237.5G  0 part 
  ├─fishserver--vg-root   254:0    0  27.9G  0 lvm  /
  ├─fishserver--vg-swap_1 254:1    0   976M  0 lvm  [SWAP]
  └─fishserver--vg-home   254:2    0 208.6G  0 lvm  /home

```

```
i
```

<!--more-->

```
目录存放：
    /music音乐
    /nav 数据库工具
```

## 没有ifconfig命令

```bash
安装包
apt install net-tools
```

# ifconfig：单网卡

```
fish@fishserver:~$ /sbin/ifconfig 
br-031d046d3ebf: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:a1:c0:98:79  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:c0ff:fee6:b1dd  prefixlen 64  scopeid 0x20<link>
        ether 02:42:c0:e6:b1:dd  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5  bytes 526 (526.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.2.17  netmask 255.255.255.0  broadcast 192.168.2.255
        inet6 fe80::2e0:b4ff:fe60:cea9  prefixlen 64  scopeid 0x20<link>
        inet6 2408:827a:3f:dd80:2e0:b4ff:fe60:cea9  prefixlen 64  scopeid 0x0<global>
        ether 00:e0:b4:60:ce:a9  txqueuelen 1000  (Ethernet)
        RX packets 244  bytes 36051 (35.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 235  bytes 34042 (33.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth22eda58: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::c079:72ff:fe92:35f7  prefixlen 64  scopeid 0x20<link>
        ether c2:79:72:92:35:f7  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16  bytes 1392 (1.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


```



# 更换源

修改文件 /etc/apt/sources.list 为下面内容

```bash
# Debian 10 buster

# 中科大源

deb http://mirrors.ustc.edu.cn/debian buster main contrib non-free
deb http://mirrors.ustc.edu.cn/debian buster-updates main contrib non-free
deb http://mirrors.ustc.edu.cn/debian buster-backports main contrib non-free
deb http://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free

deb-src http://mirrors.ustc.edu.cn/debian buster main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian buster-updates main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian buster-backports main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free

# 官方源

deb http://deb.debian.org/debian buster main contrib non-free
deb http://deb.debian.org/debian buster-updates main contrib non-free
deb http://deb.debian.org/debian-security/ buster/updates main contrib non-free

deb-src http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian-security/ buster/updates main contrib non-free

# 网易源

deb http://mirrors.163.com/debian/ buster main non-free contrib
deb http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb http://mirrors.163.com/debian/ buster-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ buster/updates main non-free contrib

deb-src http://mirrors.163.com/debian/ buster main non-free contrib
deb-src http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.163.com/debian-security/ buster/updates main non-free contrib

# 阿里云

deb http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
deb http://mirrors.aliyun.com/debian-security buster/updates main

deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.aliyun.com/debian-security buster/updates main
```

```bash
apt install sudo
apt install docker
vim /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://7bezldxe.mirror.aliyuncs.com/",
    "https://docker.mirrors.ustc.edu.cn/",
    "https://hub-mirror.c.163.com",
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [],
  "debug": false,
  "experimental": false,
  "features": {
    "buildkit": true
  }
}
```

how to install docker

```url
https://docs.docker.com/desktop/
```

```
apt install curl
```

```
 curl -fsSL https://get.docker.com -o get-docker.sh
```

```
 sudo sh get-docker.sh
```

# docker安装navidrome

## 官方的写法：

```bash
version: "3"
services:
  navidrome:
    image: deluan/navidrome:latest
    user: 1000:1000 # should be owner of volumes
    ports:
      - "4533:4533"
    restart: unless-stopped
    environment:
      # Optional: put your config options customization here. Examples:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info  
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""
    volumes:
      - "/path/to/data:/data"
      - "/path/to/your/music/folder:/music:ro"
      
      
      
      其他1；
      version: '3.8'
      services:  
      navidrome:    
      image: 'deluan/navidrome:latest'    
      container_name: navidrome        restart: unless-stopped      
      network_mode: bridge        environment:     
      - ND_SCANNER_EXTRACTOR=ffmpeg               PND_ENABLETRANSCODINGCONFIGGID=true                - ND_ENABLESHARING=true                - ND_SCANSCHEDULE=1h        ports:      - '4533:4533'    
      volumes:      - /share/Container/navidrome/data:/data            - 		/share/media2/music:/music

```



```bash
docker-compose
创建包含以下内容的文件（或添加服务 下面到您现有的文件）：docker-compose.yml
navidrome

创建一个docker-compose.yml文件,内容如下：

version: "3"
services:
  navidrome:
    image: deluan/navidrome:latest
    user: 1000:1000 # should be owner of volumes
    ports:
      - "4533:4533"
    restart: unless-stopped
    environment:
      # Optional: put your config options customization here. Examples:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info  
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""
    volumes:
      - "/home/fish/navidrome:/data"
      - "/home/fish/music:/music:ro"
```

```
docker-compose up -d
```

```bash
fish@fishserver:~$ sudo docker ps
CONTAINER ID   IMAGE                     COMMAND            CREATED          STATUS                          PORTS     NAMES
36f1b4c09f21   deluan/navidrome:latest   "/app/navidrome"   10 minutes ago   Restarting (1) 34 seconds ago             fish_navidrome_1

fish@fishserver:~$ sudo docker rm 36f1b4c09f21
Error response from daemon: You cannot remove a restarting container 36f1b4c09f21d86076b10c9d7f9659eeff2976c4199e4b3beab2f26112503b0c. Stop the container before attempting removal or force remove

fish@fishserver:~$ sudo docker rm -f  36f1b4c09f21
36f1b4c09f21

fish@fishserver:~$ 


```

## 第二种方式

```bash
docker run -d \
   --name navidrome \
   --restart=unless-stopped \
   --user $(id -u):$(id -g) \
   -v /home/fish/music:/music:ro \
   -v /home/fish/navidrome:/data \
   -p 4533:4533 \
   -e ND_LOGLEVEL=info \
   deluan/navidrome:latest
```

```bash
docker run -d -v /home/fish/music:/srv -v /home/fish/filebrowser/filebrowserconfig.json:/etc/config.json -v /home/fish/filebrowser/database.db:/etc/database.db -p 8080:80 filebrowser/filebrowser

```

# ddns-go

```bash
docker run -d --name ddns-go --restart=always --net=host -v /home/fish/ddns-go:/root jeessy/ddns-go
```

##  Docker中使用(来自官方文档)

- 不挂载主机目录, 删除容器同时会删除配置

  ```
  # host模式, 同时支持IPv4/IPv6, Liunx系统推荐
  docker run -d --name ddns-go --restart=always --net=host jeessy/ddns-go
  # 桥接模式, 只支持IPv4, Mac/Windows系统推荐
  docker run -d --name ddns-go --restart=always -p 9876:9876 jeessy/ddns-go
  ```

- 在浏览器中打开`http://主机IP:9876`，修改你的配置，成功

- [可选] 挂载主机目录, 删除容器后配置不会丢失。可替换 `/opt/ddns-go` 为主机目录, 配置文件为隐藏文件

  ```
  docker run -d --name ddns-go --restart=always --net=host -v /opt/ddns-go:/root jeessy/ddns-go
  ```

- [可选] 支持启动带参数 `-l`监听地址 `-f`间隔时间(秒)

  ```
  docker run -d --name ddns-go --restart=always --net=host jeessy/ddns-go -l :9877
  ```
