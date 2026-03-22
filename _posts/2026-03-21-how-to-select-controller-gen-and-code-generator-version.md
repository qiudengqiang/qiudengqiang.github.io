---
title: 如何选择contoller-gen和code-generator版本？
date: 2026-03-21 08:20:47
tags: [Kubernetes]
categories: k8s
---


## 版本选错会有什么问题
- code-generator 选错版本容易出现 编译错误、类型不兼容、生成代码不可用等问题。
- controller-gen 选错版本更容易出现 CRD YAML 生成结果和集群不兼容、apply 时报错、未来埋雷。

## 如何选择正确的版本

k8s，controller-gen，code-generator版本关系参考的官方资料：
- https://github.com/kubernetes/code-generator
- https://github.com/kubernetes-sigs/kubebuilder
- https://book.kubebuilder.io/versions_compatibility_supportability
- https://github.com/kubernetes-sigs/kubebuilder-release-tools/blob/master/VERSIONING.md
- https://github.com/kubernetes-sigs/controller-tools


code-generator工具集中包含了client-gen、lister-gen、informer-gen、deepcopy-gen、conversion-gen等工具

而code-generator、client-go、apimachinery都是kubernetes的staging项目中的子项目。K8s发版时会同时更新。
k8s是v1.x，他们就是v0.x，严格跟 K8s 版本一一对应。例如：
- K8s (v1.20.13) → code-generator、client-go、 apimachinery（v0.20.x）→  controller-gen (v0.5.0)
- K8s (v1.31) → code-generator、client-go、 apimachinery（v0.31.x）→  controller-gen (v0.16.0)


controller-tools 版本与 kubebuilder 版本绑定，而 kubebuilder 又与 Kubernetes 版本相关，虽然controller-gen 没有官方的“K8s版本对应表”，但可以通过它依赖的 apimachinery 版本，反推出它属于哪个 Kubernetes 时代。controller-gen 不只是要求某代K8s版本（某代指不需要关注major.minor.patch版本号中的patch），同时要求最低Go版本。

可以直接去看 controller-tools 的go.mod，其中包含k8s.io/apimachinery版本和go版本。而k8s.io/apimachinery版本是可以直接对应到k8s某一代版本的。
下面是v0.16.0版本的controller-tools的go.mod：
```go
module sigs.k8s.io/controller-tools

go 1.22.0

require (
	github.com/fatih/color v1.17.0
	github.com/gobuffalo/flect v1.0.2
	github.com/google/go-cmp v0.6.0
	github.com/onsi/ginkgo v1.16.5
	github.com/onsi/gomega v1.34.1
	github.com/spf13/cobra v1.8.1
	github.com/spf13/pflag v1.0.5
	golang.org/x/tools v0.24.0
	gopkg.in/yaml.v2 v2.4.0
	gopkg.in/yaml.v3 v3.0.1
	k8s.io/api v0.31.0
	k8s.io/apiextensions-apiserver v0.31.0
	k8s.io/apimachinery v0.31.0
	k8s.io/utils v0.0.0-20240711033017-18e509b52bc8
	sigs.k8s.io/yaml v1.4.0
)
```
可以看到，这个版本的controller-tools依赖了v0.31.0的apimachinery，对应v1.31版本的k8s。go版本最低依赖v1.22.0
