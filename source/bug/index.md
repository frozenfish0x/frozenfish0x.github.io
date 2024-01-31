***

title: bug
date: 2024-01-30 05:36:59
-------------------------

\[TOC]

# bug

## hexo：

### 2024.2.1：github问题

```
github把密码验证换成token登陆 github 连接不上不妨换成443端口试试

有时，防火墙会完全拒绝SSH连接。 如果无法使用带凭据缓存的HTTPS克隆，则可以尝试使用通过HTTPS端口建立的SSH连接进行克隆。 大多数防火墙规则应该允许这样做，但代理服务器可能会干扰。

To test if SSH over the HTTPS port is possible, run this SSH command:
要测试HTTPS端口上的SSH是否可行，请运行以下SSH命令：

\$ ssh -T -p 443 <git@ssh.github.com>

> Hi USERNAME! You've successfully authenticated, but GitHub does not
> provide shell access.

要在SSH配置文件中设置此设置，请编辑文件\~/.ssh/config，并添加以下部分：

Host github.com
Hostname ssh.github.com
Port 443
User git

```



<https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port>

## 服务器：

### 2024.1.29：防火墙

    openwrt之前的时候自动挂载出了问题，于是重新吧u盘里的系统进行重装并导入之前的备份。
    但是防火墙规则不在内，于是又是经典ipv6防火墙问题：
    防火墙wan——>     ：可以都不放行
    防火墙规则那里是优先于这个规则的
    openwrt默认是ipv4
    目标区域：如果你的想给路由器开放外网访问，那么就目标区域保持默认，如果想给路由器下的其他设备比如NAS、服务器等开放防火墙就选择lan。



    目标地址：任意就是所有的设备均可被外网访问，建议单独设置，不过默认的一般只支持ipv4，这时需要选择自定义。有两种填写方式，都是填写ipv6的后四段。均需要在末尾加入"::ffff:ffff:ffff:ffff"  如::1234:abcd:abcd:abcd/::ffff:ffff:ffff:ffff  或者  ::b64/::ffff:ffff:ffff:ffff

    或者2408::/16   or   2408::/64
    前缀为运营商。这是联通的。电信移动前缀则不同
    上此是2408/::ffff:ffff:ffff:ffff可以
    上上次是2408::/64
    这次则是2408::/16
    我的ipv6从来没变过，目前不知道怎么产生的如上问题

### 2024.1.29：配置永久变量

    debian没有ifconfig命令，安装后在/sbin/目录下。但是环境变量中没有于是配置
    vim /etc/environment
    加入如下:
    	PATH=$PATH:/sbin

    然后加载 source /etc/environment

## Linux

2024.1.30：安装fedora

```bash
1.使用他们自带的fedora media writer出现如下报错
fedora failed to start chekisomd5 after this the installation does not go any further and ihave to reset报错
使用rufus写入工具后正常

原带有gnome桌面环境。后来使用kde更换脚本，发现无法成功。
我自己进行安装kde的sddm。后来启动成功。但是出现卡顿情况
于是安装驱动。看到知乎有人说用他自带的包就好，

/sbin/lspci |grep -e VGA #显卡型号
sudo dnf update -y
sudo dnf install akmod-nvidia
sudo dnf install xorg-x11-drv-nvidia-cuda
转载自rpmfusion.org/Howto/NVIDIA

后写入kde版本出现一片空白无响应的情况。
我于是只能在原有基础上去更改成kde而不是
```

# problem

## web安全

*   [ ] &#x20;2023/12/16       fuckcdn原理，全网扫描cdn。哪天有时间可以研究一下，全网cdn扫描的源码。fuckcdn的意语言我没学过，找个c的研究，。或者去youtube或谷歌找相关技术文章。国内没有详细的讲解

