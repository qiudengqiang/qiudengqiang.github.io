---
title: 基于Kind搭建单机Kubernetes环境
date: 2026-03-17 14:01:47
tags: [Kubernetes]
categories: k8s
---

## 0.环境
配置：4c16g
系统：Ubuntu 22.04

## 1. 目标

- 单机快速搭建 Kubernetes 集群
- 支持 Operator 开发与调试
- 适配国内网络环境


## 2. 目录规划（推荐）

```bash
mkdir -p /work/{bin,src,lab/{kind,kubeadm,manifests},data}
```

```bash
/work
  /src
    /kubernetes           # k8s 源码
    /operators            # 自己的 operator 项目
    /kubebuilder-demo     # 练手工程
  /bin                    # kind/kubebuilder/kustomize/helm 等二进制
  /lab
    /kind                 # kind 配置文件
    /kubeadm              # kubeadm 配置文件
    /manifests            # 练习 yaml
  /data
    /registry             # 本地 registry 数据
    /containerd
    /docker
```

## 3.安装 Docker

```bash
apt update
apt install -y docker.io
systemctl enable --now docker
```
配置镜像加速：

```bash 
mkdir -p /etc/docker

cat > /etc/docker/daemon.json <<'EOF'
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
    "https://docker.m.daocloud.io"
  ]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

## 4.安装kubectl

```bash
cd /work/bin
curl -fLO https://dl.k8s.io/release/v1.29.2/bin/linux/amd64/kubectl
chmod +x kubectl
ln -sf /work/bin/kubectl /usr/local/bin/kubectl
```

验证
```bash 
kubectl version
```

## 5.安装Kind

```bash
cd /work/bin
curl -fLO https://dl.k8s.io/release/v1.29.2/bin/linux/amd64/kubectl
chmod +x kubectl
ln -sf /work/bin/kubectl /usr/local/bin/kubectl
```

加入path
```bash
echo 'export PATH=/work/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

验证
```bash 
kind version
```

## 6.拉取国内kind node镜像（国内加速）
```bash
docker pull m.daocloud.io/docker.io/kindest/node:v1.29.2
docker tag m.daocloud.io/docker.io/kindest/node:v1.29.2 kindest/node:v1.29.2
```

## 7.创建kind集群配置
创建kind.yaml
```bash
vim /work/lab/kind/kind.yaml
```
写入配置
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: dev
nodes:
  - role: control-plane
    image: kindest/node:v1.29.2
  - role: worker
    image: kindest/node:v1.29.2
  - role: worker
    image: kindest/node:v1.29.2
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
    endpoint = ["https://k8s.m.daocloud.io"]
```

## 8.创建集群

```bash
cd /work/lab/kind
kind create cluster --config kind.yaml
```

## 9.验证集群

```bash
kubectl get nodes
kubectl get pods -A
```
期望结果：
- 所有节点 Ready
- kube-system 下组件全部 Running

## 10.常用操作
删除集群：
```bash
kind delete cluster --name dev
```

查看日志：
```bash
kind export logs ./kind-logs
```
