title: Manjaro自动挂载samba文件夹
author: 大丈夫没问题
tags:
  - samba
  - manjaro
categories:
  - linux
date: 2019-05-04 22:03:00
---
Manjaro从地址栏访问samba共享的方式总有点问题，会时不时提示没有文件夹修改权限。最近有时间研究了下，发现命令行挂载的方式比较靠谱。

由于路由器刷的是`openwrt`，软件源里安装的samba服务端版本似乎是比较老的版本，`manjaro`命令行挂载似乎用的是较新的samba协议，调了半天发现客户端这边要加下版本号。

挂载命令：

```
sudo mount.cifs //192.168.1.1/share0 /mnt/share0 -o user=xxxx,pass=xxxx,uid=1000,gid=1000,sec=ntlmssp,vers=1.0 --verbose
```

命令行挂载调通后，要实现开机自动挂载就容易的多了，官方文档提供的多种的挂载方式，这里以`systemd`为例：

1. 在`/etc/samba/`下建立`credentials`文件夹：`sudo mkdir credentials`，创建比如名为`share0`的文件： `sudo vim share0`，内容如下：

```
   username=your samba username
   password=your samba password
```

2. 创建systemd unit文件：`sudo /etc/systemd/system/mnt-share0.mount`，内容如下：

   ```
   [Unit]
   Description=Mount Share at boot
   
   [Mount]
   What=//192.168.1.1/share0
   Where=/mnt/share0
   Options=x-systemd.automount,credentials=/etc/samba/credentials/share0,iocharset=utf8,uid=1000,gid=1000,sec=ntlmssp,vers=1.0,rw
   Type=cifs
   TimeoutSec=30
   ForceUnmount=true
   
   [Install]
   WantedBy=multi-user.target
   
   ```

   Note：这里有一点要注意的是你的挂载路径（`where`）必须与你的文件名（`mnt-share0`）对应，比如你挂载到`/mnt/share`， 那你的文件就必须为`mnt-share0.mount`

3. 启用服务`sudo systemctl enable mnt-share0.mount`
4. 启动服务`sudo systemctl start mnt-share0.mount`

到这里samba文件夹就能开机自动挂载，官方还提供了其他的自动挂载方式，详情可参考文档。






参考：

[<https://wiki.archlinux.org/index.php/samba#As_systemd_unit>](<https://wiki.archlinux.org/index.php/samba#As_systemd_unit>)