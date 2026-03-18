---
title: 使用Kubebuilder创建Operator
date: 2026-03-18 08:20:47
tags: [Kubernetes]
categories: k8s
---

## 1. 目标

- 创建 CRD（自定义资源）
- 创建 Controller（调谐逻辑）
- 本地运行调试（make run）
- 明确工具链（controller-gen）

## 2. 安装 Go（>=1.23）

```bash
cd /work/src
wget https://mirrors.aliyun.com/golang/go1.23.7.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.23.7.linux-amd64.tar.gz
```

配置环境变量:
```bash
cat >> ~/.bashrc <<'EOF'
export GOROOT=/usr/local/go
export GOPATH=/work/go
export PATH=$GOROOT/bin:$GOPATH/bin:/work/bin:$PATH
EOF

source ~/.bashrc
```

## 3.安装kubebuilder 
```bash
cd /work/bin
curl -fLo kubebuilder https://files.m.daocloud.io/github.com/kubernetes-sigs/kubebuilder/releases/download/v4.5.2/kubebuilder_linux_amd64
chmod +x kubebuilder
kubebuilder version
```

## 4.安装 controller-gen

作用：
- 生成 CRD（make manifests 依赖）
- 解析 kubebuilder 注解
```bash
GOBIN=/work/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.16.5
controller-gen --version
```

## 5.创建Operator工程
```bash
cd /work/src/operators
mkdir -p guestbook-operator
cd guestbook-operator

go mod init alphabethub.com/guestbook-operator
kubebuilder init --domain alphabethub.com --repo alphabethub.com/guestbook-operator
```

## 6.创建API(CRD+Controller)
```bash
kubebuilder create api --group apps --version v1alpha1 --kind Guestbook --resource --controller
```

## 7.生成代码
```bash
make manifests
make generate
```
## 8.安装CRD
```bash
make install
```
验证
```bash
kubectl get crd | grep guestbook
```

## 9.本地运行controller
```bash
make run
```
说明：
- 使用当前 kubeconfig（kind）
- 不需要构建镜像
- 最适合开发调试

## 10.创建资源（触发Reconcile)
```bash
kubectl apply -f config/samples/apps_v1alpha1_guestbook.yaml
kubectl get guestbooks -A
```
## 11.验证
观察controller日志，确认Watch 生效，Reconcile 正常执行，Operator 工作正常

## 12.调试建议
终端1（controller）：
```bash
make run
```
终端2（资源操作）：
```bash
kubectl apply -f ...
kubectl get guestbooks -A
```
终端3（观察集群）：
```bash
kubectl get pods -A
kubectl get events -A
```
## 13.可选工具
envtest（本地 API Server 测试）
```bash
GOBIN=/work/bin go install sigs.k8s.io/controller-runtime/tools/setup-envtest@latest
```
ginkgo(测试框架)
```bash
GOBIN=/work/bin go install github.com/onsi/ginkgo/v2/ginkgo@latest
```



