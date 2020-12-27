title: macOS安装配置Podman 2.x
author: 大丈夫没问题
tags:
  - linux
  - podman
  - docker
categories:
  - linux
date: 2020-12-27 20:16:00
---
macOS环境下的podman只是客户端，server端实际运行在linux环境下，所以需要安装linux虚拟机。

## 安装

```
brew install virtualbox
brew install vagrant
brew install podman
```

## 配置虚拟机

新建个文件夹，里面创建`Vagrantfile`文件，内容如下：

```
Vagrant.configure("2") do |config|
  config.vm.box = "generic/fedora32"
  config.vm.hostname = "fedora"
  config.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
     vb.cpus = 1
  end
end
```

### 启动虚拟机
vagrant up

### 配置fedora

安装podman server端

```
sudo dnf -y install podman
systemctl --user enable --now podman.socket
sudo loginctl enable-linger $USER
sudo systemctl enable --now sshd
```

验证安装是否成功

```
podman --remote info
```

### 查看虚拟机ssh配置

`vagrant ssh-config`

## podman客户端配置

添加连接

podman system connection add baude --identity /Users/chen/vm/fedora/.vagrant/machines/default/virtualbox/private_key ssh://vagrant@127.0.0.1:2222/run/user/1000/podman/podman.sock

## 替换docker

```
alias docker=podman
```

## 参考
* [Podman Remote clients for MacOS and Windows](https://github.com/containers/podman/blob/master/docs/tutorials/mac_win_client.md)