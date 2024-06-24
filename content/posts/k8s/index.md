---
title: "k8s"
summary: Learn About All Features in PaperMod
date: 2024-06-24
weight: 1
tags: ["Web3"]
author: ["Evanara Kumestra"]
---

# Linux

BIOS - MBR/EFI - boot loader - Linux

kcna learning path

https://training.linuxfoundation.org/wp-content/uploads/2023/07/KCNA_CurriculumPath_Jul23.pdf

**Labels**: Labels are key-value pairs attached to Kubernetes objects such as Pods, ReplicaSets, Nodes, Namespaces and Persistent Volumes.


# Component

## Pod

- Smallest unit of K8s
- Abstraction over container
- Usually 1 application per Pod
- Each Pod gets its own IP address
- New IP address on re-creation

## Workload

https://kubernetes.io/docs/concepts/workloads/

## Service

A Service offers a single DNS entry for a stateless containerized application managed by the Kubernetes cluster, regardless of the number of its replicas, by providing a common load balancing access point to a set of pods logically grouped and managed by a controller such as a Deployment, ReplicaSet, or DaemonSet.

## Namespace

https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

# Architecture

## Node

- Kubelet
- Kube Proxy
- Container Runtime

## Master

- API Server
- Scheduler
- Controller Manager
- etcd

# Demo

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

安装minikube,安装方式参考链接 https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download

```shell {linenos=true}
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

在`~./bashrc`中加上如下一行：

```shell {linenos=true}
alias kubectl="minikube kubectl --"
```

现在创建一个demo，由mongo-express，mongodb构成。先创建一个secret，这个secret中含有mongodb的账号密码。

```shell {linenos=true}
kubectl apply -f mongo-secret.yaml
```

```yaml {linenos=true}
apiVersion: v1
kind: Secret
metadata:
    name: mongodb-secret
type: Opaque
data:
    mongo-root-username: dXNlcm5hbWU=
    mongo-root-password: cGFzc3dvcmQ=
```

然后创建一个mongodb的deployment和service，mongodb需要的账号密码从之前创建的secret里面读取。

```shell {linenos=true}
kubectl apply -f mongo.yaml
```

```yaml {linenos=true}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

然后创建一个configmap，这个configmap包含mongodb service的链接地址，然后创建mongodb express service，这个service从configmap中读取mongodb所在位置url。

```shell {linenos=true}
kubectl apply -f mongo-configmap.yaml
```

```yaml {linenos=true}
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```

```shell {linenos=true}
kubectl apply -f mongo-express.yaml
```

```yaml {linenos=true}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom: 
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer  
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000
```