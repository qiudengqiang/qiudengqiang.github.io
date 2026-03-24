---
title: 用controller-gen和code-generator为已有CRD新增字段
date: 2026-03-24 08:20:47
tags: [Kubernetes]
categories: k8s
---


## controller-gen

### 生成 deepcopy
```go
cd /work/src/operators/guestbook-operator

rm -f api/apps/v1alpha1/zz_generated.deepcopy.go

/work/bin/controller-gen \
  object:headerFile="hack/boilerplate.go.txt" \
  paths="./api/..."
```
验证：
```go
cd /work/src/operators/guestbook-operator
ls -l api/apps/v1alpha1/zz_generated.deepcopy.go
```

### 生成 crd yaml
```bash
rm -f config/crd/v1alpha1/*.yaml
controller-gen crd paths="./api/..." output:crd:artifacts:config=config/crd/v1alpha1
```
验证：
```go
cd /work/src/operators/guestbook-operator
ls -l config/crd/v1alpha1/apps.alphabethub.com_guestbooks.yaml
```


## code-generator

### 前提
确认类型上有 +genclient
```go
// +genclient
// +kubebuilder:object:root=true
// +kubebuilder:subresource:status
type Guestbook struct {
```

确认类型List上方要有
```go
// +kubebuilder:object:root=true
type GuestbookList struct {
```

### 准备临时 GOPATH
```bash
rm -rf .gopath
mkdir -p .gopath/src/alphabethub.com
```
然后建立软链接：
```bash
cd /work/src/operators/guestbook-operator
ln -sfn "$(pwd)" .gopath/src/alphabethub.com/guestbook-operator
```
验证软链存在：
```bash
ls -l .gopath/src/alphabethub.com
```

### 下载code-generator（若不存在）

```bash
cd /work/src
git clone --depth=1 --branch=v0.29.2 https://github.com/kubernetes/code-generator.git
```

### 清理旧产物
```bash
cd /work/src/operators/guestbook-operator
rm -rf pkg/client
mkdir -p pkg/client
```

### 生成 client/lister/informer
```bash
cd /work/src/operators/guestbook-operator

GOPATH=/work/src/operators/guestbook-operator/.gopath \
/work/src/code-generator/generate-groups.sh \
  "client,lister,informer" \
  alphabethub.com/guestbook-operator/pkg/client \
  alphabethub.com/guestbook-operator/api \
  "apps:v1alpha1" \
  --go-header-file "$(pwd)/hack/boilerplate.go.txt" \
  --output-base "$(pwd)/.gopath/src"
```
### 检查临时 GOPATH 里的真实生成结果
需要包含clientset,
```bash
find .gopath/src/alphabethub.com/guestbook-operator/pkg/client -type f | sort
find ./pkg/client -type f | sort
```
验证是否包含以下文件：
```bash
.../typed/apps/v1alpha1/guestbook.go
.../typed/apps/v1alpha1/fake/fake_guestbook.go
.../listers/apps/v1alpha1/guestbook.go
.../informers/externalversions/apps/v1alpha1/guestbook.go
```

### 删除.gopath
```bash
cd /work/src/operators/guestbook-operator

rm -rf .gopath
```

### 编译
执行

```bash
# go build ./...
```
这时候大概率是会报错的
```bash
# alphabethub.com/guestbook-operator/pkg/client/clientset/versioned/typed/apps/v1alpha1
pkg/client/clientset/versioned/typed/apps/v1alpha1/apps_client.go:87:17: undefined: v1alpha1.SchemeGroupVersion
# alphabethub.com/guestbook-operator/pkg/client/listers/apps/v1alpha1
pkg/client/listers/apps/v1alpha1/guestbook.go:95:43: undefined: v1alpha1.Resource
```
code-generator 生成出来的 typed client / lister 默认会依赖这两个符号：
- SchemeGroupVersion
- Resource(...)
- Kind(...)


但 Kubebuilder 默认生成的 groupversion_info.go（register.go） 里通常只有：
- GroupVersion
- SchemeBuilder
- AddToScheme

所以需要修改 groupversion_info.go :
```go
package v1alpha1

import (
	"k8s.io/apimachinery/pkg/runtime/schema"
	"sigs.k8s.io/controller-runtime/pkg/scheme"
)

var (
	// GroupVersion is group version used to register these objects.
	GroupVersion = schema.GroupVersion{Group: "apps.alphabethub.com", Version: "v1alpha1"}

	// SchemeGroupVersion is an alias used by generated clientset/lister/informer code.
	SchemeGroupVersion = GroupVersion

	// SchemeBuilder is used to add go types to the GroupVersionKind scheme.
	SchemeBuilder = &scheme.Builder{GroupVersion: GroupVersion}

	// AddToScheme adds the types in this group-version to the given scheme.
	AddToScheme = SchemeBuilder.AddToScheme
)

// Resource takes an unqualified resource and returns a Group qualified GroupResource.
func Resource(resource string) schema.GroupResource {
	return SchemeGroupVersion.WithResource(resource).GroupResource()
}

// Kind takes an unqualified kind and returns a Group qualified GroupKind.
func Kind(kind string) schema.GroupKind {
	return SchemeGroupVersion.WithKind(kind).GroupKind()
}
```


## apply 
### apply crd
kubectl apply -f config/crd/v1alpha1/apps.alphabethub.com_guestbooks.yaml
kubectl get crd guestbooks.apps.alphabethub.com -oyaml|less

### apply cr
kubectl apply -f config/samples/apps_v1alpha1_guestbook.yaml
kubectl get guestbook guestbook-sample -oyaml|less

