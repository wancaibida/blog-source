title: Openwrt x86安装教程
author: 大丈夫没问题
tags:
  - openwrt
  - linux
categories:
  - linux
date: 2019-03-13 20:12:00
---
网上Openwrt x86虚拟机安装的教程一大堆，却很少见到非虚拟机的教程，现在正好有机会重新刷机，就顺便把步骤记录一下，免得再次踩坑。

## 镜像下载

第一步就是下载openwrt x86的镜像，这里两个镜像可以选择，带`squashfs`的刷机后会有一个只读的分区，带`ext4`的所有分区都是可读写的。本教程是以`combined-ex4`为例子的。

下载地址：[openwrt-18.06.2-x86-64-combined-ext4.img.gz](https://downloads.openwrt.org/releases/18.06.2/targets/x86/64/openwrt-18.06.2-x86-64-combined-ext4.img.gz)

> ### combined-squashfs 和 combined-ex4 的区别：

> **combined-squashfs.img.gz** This disk image uses the traditional OpenWrt layout, a squashfs read-only root filesystem and a read-write partition where settings and packages you install are stored. Due to how this image is assembled, you will have only 230-ish MB of space to store additional packages and configuration, and Extroot does not work.
  
> **combined-ext4.img.gz** This disk image uses a single read-write ext4 partition with no read-only squashfs root filesystem, which allows to enlarge the partition. Features like Failsafe Mode or Factory Reset won't be available as they need a read-only squashfs partition to function.



## 制作linux启动盘

我们需要两个U盘，其中一个就是用来做linux启动盘，这里以[SystemRescueCd](http://www.system-rescue-cd.org/)为例，制作USB启动盘的教程可一参考这里：[Installing-SystemRescueCd-on-a-USB-stick/](http://www.system-rescue-cd.org/Installing-SystemRescueCd-on-a-USB-stick/)

USB启动盘制作好后，将下载下来的openwrt安装包拷贝到另一个U盘。

## 安装镜像

将两个U盘插入软路由，以linux启动盘启动，成功进入系统后执行以下命令将第二个U盘挂载到系统里：

```
mkdir /mnt/usb
mount /dev/sdx /mnt/usb
```

解压并将镜像刷到软路由内存里：

```
cd /mnt/usb
gunzip openwrt-18.06.2-x86-64-combined-ext4.img.gz
dd if=openwrt-18.06.2-x86-64-combined-ext4.img of=/dev/sdX
```

其中`sdx`就是你软路由的安装盘。

PS：官网里的步骤并没有提到解压这一步骤，实际操作中直接`dd`的话，openwrt是无法启动的，可能是linux系统的差异。

### 分区扩容（可选）

刷入openwrt后，执行`fdisk -l`，就会看到软路由的`sdx`盘被分成了两个分区，你会发现第二个分区只有256M，这里就可以调节第二个分区的大小，将硬盘的剩余容量全部划分到第二个分区：

1. `fdisk -l /dev/sdx`将第二个分区的起始扇区号`start`的值记下来
2. `fdisk /dev/sdx` 进入修改硬盘分区信息
3. 输入`d`，然后回车
4. 这里会提示你要删除哪个分区，输入`2`，然后回车
5. 输入`n`，然后回车，选择分区类型`primary`或`extended`都可以。
6. 选择分区后会让你输入分区开始扇区号，这里输入之前记下来的扇区号
7. 输入`w`，这时候会提示你`Partition #2 contains a ext4 signature. Do you want to remove the signature?`，**这一步很重要**，这里要选`no`
8. 输入`w`，然后回车
9. 分区完成后执行`resize2fs /dev/sdx2`，分区扩容就大功告成了。

## 系统配置
* 重启进入Openwrt系统后，系统会开放一个lan口，将网线接入这个lan口后就可以通过`192.168.1.1`来访问路由器管理界面了。
* 进入管理界面后需要设置管理员密码，同时你可以上传`ssh key`，免得每次`ssh`都需要输入密码。
* 在`interfaces`页面，`LAN`标签页面，在`Physical settings`->`Interface`你可以修改路由的lan口，将另外几个没开放的lan口都勾选。
* 在`WAN`标签页面，`Protocol`选择`PPPoE`，输入宽带密码，`save and apply`就可以上网了。

## 应用安装
我们会安装一些应用，但这里有个坑就是应用的安装源是`http`的，可能会被劫持，所以要修改成https的。先执行以下命令，安装必备的包：

```
opkg update
opkg install ca-certificates ca-bundle
opkg install libustream-mbedtls
opkg install git-http curl 
```

修改源为`https`

```
//将http改为https
vim /etc/opkg/distfeeds.conf
opkg update
//安装一些调试用的软件
opkg install bind-dig
```


## 一些常见问题

1. Samba添加用户与设置密码
```
vi /etc/passwd
smbpasswd -a samba
```
2. `dnsmasq`和`dnsmasq-full`的区别

    `dnsmasq-full`多了`ipset`功能

3. `dnsmqsq --no-resolv`
```
    --no-resolv
    Don't read /etc/resolv.conf. Get upstream servers only from the command line or the dnsmasq configuration file.
```

4. `ss-tunnel` dns解析不了

  一般原因有以下几点：

    1. `ss-tunnel` 未加上`-u`参数，开启`udn relay`

    2. `ss-server` 未加上`-u`参数，开启`udn relay`

    3. 服务端未开启端口

   调试用的命令：
```
        //dnstracer
        dnstracer -vo  -s 127.0.0.1 google.com
        
        //dns测试
        dig +trace google.com 
        // dns请求也可以直接发送给ss-tunnel
        dig google.com @127.0.0.1 -p 5353
```

5. 开启`ss-rules`后域名无法解析

  具体什么原因不是很清楚，可能是`iptables`对`udp`转发有点问题，`ss-redir for UDP`这项打勾去掉即可。

## 一些定时任务脚本

写了一些定时[任务脚本](https://github.com/wancaibida/openwrt-scripts)，这里记录一下。

## 安全设置

### Luci页面访问控制

Openwrt的管理页面默认是允许任何IP访问的，如果你的路由器有公网IP的话，那任何人都可访问到你家的路由器登录页面，通过下面设置只允许来自内网地址的访问：

修改配置文件：
```
vi /etc/config/uhttpd
```

将`192.168.1.1`和`fd54:274c:96dd::1`换成路由器的v4和v6地址，路由器的lan IP可以在`Network`-> `Interfaces`->`LAN`查看，openwrt默认情况下是`192.168.1.1`。
```
	# HTTP listen addresses, multiple allowed
	list listen_http	192.168.1.1:80
	list listen_http	[fd54:274c:96dd::1]:80

	# HTTPS listen addresses, multiple allowed
	list listen_https	192.168.1.1:443
	list listen_https	[fd54:274c:96dd::1]:443
```

*Note*: 如果要访问通过IPv6访问界面，URL应该是这个样子的：`http://[fd54:274c:96dd::1]`

修改完成后重启服务：`/etc/init.d/uhttpd restart`

### SSH 访问控制

登录openwrt管理页面 `System` -> `Administration`->`Dropbear Instance`

`Interface` 那一项选择`lan`，然后`Save & Apply`，这样你的路由器ssh只能通过内网访问了。



## 参考


* [https://openwrt.org/docs/guide-user/installation/openwrt_x86](https://openwrt.org/docs/guide-user/installation/openwrt_x86)
* [dnsmasq-man.html](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html)
* [openwrt 18.06.1 配置科学上网](https://medium.com/@cnnbysy/openwrt-18-06-1-配置科学上网-30e231958c38)
* [OpenWrt $$ 安装&配置指南](https://alalin.me/archives/805)
* [那些从墙上学会的知识](https://icymind.com/learn-from-gfw/)
* [samba 配置](https://openwrt.org/docs/guide-user/services/nas/samba_configuration)

## 更新日志

* 2019-11-09 添加了安全设置