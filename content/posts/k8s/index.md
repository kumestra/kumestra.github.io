---
title: "k8s"
summary: Learn About All Features in PaperMod
date: 2024-06-24
weight: 1
tags: ["Web3"]
author: ["Evanara Kumestra"]
---

* Pod

- Smallest unit of K8s
- Abstraction over container
- Usually 1 application per Pod
- Each Pod gets its own IP address
- New IP address on re-creation

要想使用k8s，需要先安装Docker Engine，安装指南在 https://docs.docker.com/engine/install/ubuntu/，这里面列出了几种安装方式，我采用下载deb安装包的方式进行安装。今天（2024年6月24日）最新的安装包如下，执行安装命令，第一次安装失败了，提示没有安装iptables，安装iptables之后再次安装成功。

```shell {linenos=true}
sudo dpkg -i ./containerd.io_1.6.33-1_amd64.deb ./docker-ce_26.1.4-1~ubuntu.24.04~noble_amd64.deb docker-ce-cli_26.1.4-1~ubuntu.24.04~noble_amd64.deb docker-buildx-plugin_0.14.1-1~ubuntu.24.04~noble_amd64.deb docker-compose-plugin_2.27.1-1~ubuntu.24.04~noble_amd64.deb
```

在安装完成之后，要想使用docker需要`sudo`，可以将当前的用户添加到`docker`用户组，这样当前用户可以直接使用docker

```shell {linenos=true}
sudo groupadd docker
sudo usermod -aG docker $USER
```

minikube需要至少20g的闲置磁盘空间，需要提前准备好（磁盘，分区，物理卷，卷组，逻辑卷）