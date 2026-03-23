---
title: 线上K8s CRD 新增字段指南（无业务语义版）
date: 2026-03-23 08:20:47
tags: [Kubernetes]
categories: k8s
---

## 1. 目标

本文总结一个可复用的工程流程：在 Kubernetes CRD 中新增字段，并确保代码生成、清单产物、集群校验、版本一致性全部可控。

---

## 2. 总体原则

1. `types.go` 改动后，生成器流程必须重跑（即使部分产物最终无 diff）。
2. 生成器版本必须固定，避免不同开发机生成结果漂移。
3. 声明式清单只保留期望态配置，避免提交服务端运行态字段。
4. 验证要同时覆盖“代码侧”和“集群侧”。

---

## 3. 字段定义层（API Types）

### 3.1 选择字段位置

- 配置型字段放 `spec`。
- 观测/回显型字段放 `status`。
- 如果需要“实际生效值”可观测，可在 `status` 增加同名回显字段。

### 3.2 定义 enum（示例）

```go
type PolicyType string

const (
    PolicyA PolicyType = "PolicyA"
    PolicyB PolicyType = "PolicyB"
)

type ExampleSpec struct {
    // +kubebuilder:validation:Enum=PolicyA;PolicyB
    Policy PolicyType `json:"policy,omitempty"`
}
```

### 3.3 为什么有些字段不会改动 `deepcopy` 文件

- 值类型（如 `string/int/bool` 及其别名）通常由 `*out = *in` 覆盖，不一定产生新代码。
- 引用类型（`slice/map/pointer`）通常会生成单独拷贝逻辑（`make+copy` 或递归拷贝）。

结论：**必须重跑 `deepcopy-gen`，但不保证一定有 diff。**

---

## 4. 代码生成层（必须执行）

建议统一跑完整链路：

1. `deepcopy-gen`
2. `conversion-gen`
3. `client-gen`
4. `lister-gen`
5. `informer-gen`
6. `controller-gen`（CRD YAML）

原因：

- 可以避免“以为不用重跑，实际遗漏”的风险。
- 即使 `clientset/lister/informer` 无变更，也应由流程证明“已重跑且无变化”。

---

## 5. 版本锁定与可复现

## 5.1 固定工具版本

- 固定 `code-generator` 版本。
- 固定 `controller-gen` 版本。

## 5.2 本地工具目录

将二进制安装到仓库内目录（如 `.tools/bin`），不要依赖全局 `$GOPATH/bin`。

## 5.3 按版本校验后再安装

不要只判断“文件是否存在”，要校验模块版本：

```bash
go version -m <bin> | awk '$1=="mod"{print $2"@"$3; exit}'
```

若不匹配目标版本，再 `go install ...@<version>`。

## 5.4 老版本工具与新 Go 兼容性

某些老版本生成器在新 Go 工具链上可能异常，可单独指定：

```bash
GOTOOLCHAIN=go1.xx.yy go install <tool>@<version>
```

---

## 6. 仅更新目标 CRD 的安全策略

如果仓库含多个 CRD，但只希望更新其中一个：

1. `controller-gen` 先输出到临时目录。
2. 只拷贝目标 CRD 文件回仓库。
3. 严格检查其余 CRD 文件无 diff。

这能避免“全量重写所有 CRD”造成无关变更。

---

## 7. CRD 清单净化（建议）

生成后应去掉不需要提交的字段：

- 顶层 `status:`（CRD 对象运行态，由 apiserver 回填）
- `metadata.creationTimestamp: null`

保留：

- `spec.versions[*].subresources.status: {}`（这是 CRD 能力配置，不是运行态）

---

## 8. 验证清单（Checklist）

### 8.1 代码与产物

1. `git diff --name-only` 仅包含预期文件。
2. 目标字段出现在 `types` 与目标 CRD schema。
3. 非目标 CRD 不应被修改（如需求要求“只改一个 CRD”）。

### 8.2 编译/测试

1. `go test ./...`（至少保证编译通过）。
2. 如有 CI 生成一致性检查，确保通过。

### 8.3 集群侧校验

优先：

```bash
kubectl apply --dry-run=server -f <crd.yaml>
```

若集群不支持 `--server-side`（报 `415 Unsupported Media Type`），属于集群能力问题，可改用上面这条普通 server dry-run 校验。

---

## 9. 常见误区

1. **误区：改了 types，clientset/lister/informer 一定有 diff。**  
   实际：很多字段变更不会改变这些生成文件内容。

2. **误区：本地有工具二进制就算锁版本。**  
   实际：可能是旧版本，必须做版本校验。

3. **误区：把 CRD 顶层 `status` 一起提交更完整。**  
   实际：这是运行态信息，通常不应进入声明式清单。

---

## 10. 最小执行模板（示意）

```make
# 1) tools: 校验版本，不匹配则安装
# 2) codegen-all: 跑 deepcopy/conversion/client/lister/informer
# 3) crd-target: controller-gen 到临时目录，仅拷贝目标 CRD
# 4) post-process: 去掉顶层 status + creationTimestamp
# 5) verify: diff 范围与 go test
```

把以上流程固化为 Make 目标后，可以将“新增 CRD 字段”变成稳定、可审计、可复现的一步式操作。
