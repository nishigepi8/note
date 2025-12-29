# kubectl 命令速查手册

## 前言

Kubernetes 作为容器编排的事实标准，其命令行工具 `kubectl` 是日常运维和开发中最重要的工具。本文整理了我在实际工作中最常用的 kubectl 命令，并附上使用场景和最佳实践，希望能帮助大家提高工作效率。

## 基础概念回顾

在开始之前，先简单回顾一下 Kubernetes 的核心资源：

- **Pod**：最小的部署单元，包含一个或多个容器
- **Service**：为 Pod 提供稳定的网络访问入口
- **Deployment**：管理 Pod 的副本和更新策略
- **Ingress**：提供 HTTP/HTTPS 路由规则

## Pod 操作命令

Pod 是 Kubernetes 中最基础的资源，日常排查问题、查看日志都离不开它。

### 查看 Pod 列表

```bash
# 查看指定 namespace 下的所有 Pod
kubectl get pod -n ${namespace}

# 查看所有 namespace 的 Pod
kubectl get pods --all-namespaces
# 或简写
kubectl get pods -A

# 查看默认 namespace（default）的 Pod
kubectl get pods
```

**使用场景**：
- 快速查看 Pod 运行状态
- 检查 Pod 是否正常启动
- 排查 Pod 数量是否符合预期

### 查看 Pod 详细信息

```bash
# 查看 Pod 的详细信息（包括 IP、节点、事件等）
kubectl get pod ${pod-name} -o wide

# 查看 Pod 的完整 YAML 配置
kubectl get pod ${pod-name} -o yaml

# 查看 Pod 的详细描述（推荐，包含事件和状态信息）
kubectl describe pod ${pod-name} -n ${namespace}
```

**使用场景**：
- 排查 Pod 启动失败问题（`describe` 命令会显示事件）
- 查看 Pod 分配到的节点
- 检查 Pod 的资源配置

### Pod 日志查看

```bash
# 查看 Pod 日志
kubectl logs ${pod-name} -n ${namespace}

# 实时跟踪日志（类似 tail -f）
kubectl logs -f ${pod-name} -n ${namespace}

# 查看最近 100 行日志
kubectl logs --tail=100 ${pod-name} -n ${namespace}

# 查看多容器 Pod 中特定容器的日志
kubectl logs ${pod-name} -c ${container-name} -n ${namespace}
```

**使用场景**：
- 排查应用错误
- 监控应用运行状态
- 调试部署问题

### Pod 调试命令

```bash
# 进入 Pod 容器执行命令（类似 docker exec）
kubectl exec -it ${pod-name} -n ${namespace} -- /bin/bash

# 在 Pod 中执行单条命令
kubectl exec ${pod-name} -n ${namespace} -- ls -la

# 删除 Pod（会触发重建，如果由 Deployment 管理）
kubectl delete pod ${pod-name} -n ${namespace}
```

**使用场景**：
- 调试容器内部问题
- 手动执行维护操作
- 强制重启 Pod（删除后自动重建）

## Service 操作命令

Service 为 Pod 提供稳定的访问入口，是服务发现的核心组件。

### 查看 Service 列表

```bash
# 查看指定 namespace 下的 Service
kubectl get svc -n ${namespace}

# 查看所有 Service
kubectl get services --all-namespaces

# 查看 Service 的详细信息（包括 Endpoints）
kubectl get svc ${service-name} -n ${namespace} -o wide
```

**使用场景**：
- 检查服务是否正常暴露
- 查看 Service 的 ClusterIP 和端口
- 验证服务发现配置

### 查看 Service 详细信息

```bash
# 查看 Service 的详细描述（推荐）
kubectl describe svc ${service-name} -n ${namespace}

# 查看 Service 的完整 YAML 配置
kubectl get svc ${service-name} -n ${namespace} -o yaml

# 查看 Service 对应的 Endpoints（后端 Pod）
kubectl get endpoints ${service-name} -n ${namespace}
```

**使用场景**：
- 排查服务无法访问问题
- 检查 Service 是否正确关联到 Pod
- 验证负载均衡配置

### Service 调试

```bash
# 临时端口转发（用于本地调试）
kubectl port-forward svc/${service-name} 8080:80 -n ${namespace}

# 查看 Service 的 Endpoints 详情
kubectl describe endpoints ${service-name} -n ${namespace}
```

**使用场景**：
- 本地测试服务功能
- 排查网络连接问题
- 验证服务配置

## Deployment 操作命令

Deployment 是管理应用部署的核心资源，支持滚动更新、回滚等高级功能。

### 查看 Deployment 列表

```bash
# 查看指定 namespace 下的 Deployment
kubectl get deploy -n ${namespace}

# 查看所有 Deployment
kubectl get deployments --all-namespaces

# 查看 Deployment 的详细信息
kubectl get deploy ${deployment-name} -n ${namespace} -o wide
```

### 查看 Deployment 详细信息

```bash
# 查看 Deployment 的详细描述（推荐）
kubectl describe deploy ${deployment-name} -n ${namespace}

# 查看 Deployment 的完整 YAML 配置
kubectl get deploy ${deployment-name} -n ${namespace} -o yaml
```

### Deployment 更新操作

```bash
# 更新 Deployment 的镜像版本
kubectl set image deployment/${deployment-name} ${container-name}=${new-image} -n ${namespace}

# 查看 Deployment 的更新历史
kubectl rollout history deployment/${deployment-name} -n ${namespace}

# 回滚到上一个版本
kubectl rollout undo deployment/${deployment-name} -n ${namespace}

# 回滚到指定版本
kubectl rollout undo deployment/${deployment-name} --to-revision=${revision-number} -n ${namespace}

# 查看滚动更新状态
kubectl rollout status deployment/${deployment-name} -n ${namespace}
```

**使用场景**：
- 日常应用更新
- 快速回滚到稳定版本
- 查看更新历史记录

### Deployment 扩缩容

```bash
# 扩容到指定副本数
kubectl scale deployment/${deployment-name} --replicas=${replica-count} -n ${namespace}

# 查看当前副本数
kubectl get deploy ${deployment-name} -n ${namespace}
```

**使用场景**：
- 应对流量高峰
- 资源优化调整
- 测试高可用性

### Deployment 删除

```bash
# 删除 Deployment（会同时删除关联的 Pod）
kubectl delete deployment ${deployment-name} -n ${namespace}

# 删除 Deployment 但保留 Pod（不常用）
kubectl delete deployment ${deployment-name} --cascade=false -n ${namespace}
```

## Ingress 操作命令

Ingress 提供 HTTP/HTTPS 路由规则，是外部访问集群服务的主要方式。

### 查看 Ingress 列表

```bash
# 查看指定 namespace 下的 Ingress
kubectl get ingress -n ${namespace}

# 查看所有 Ingress
kubectl get ingress --all-namespaces

# 查看 Ingress 的详细信息
kubectl describe ingress ${ingress-name} -n ${namespace}
```

### Ingress 配置示例

以下是一个完整的 Ingress 配置示例，适用于 Nginx Ingress Controller：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress 
metadata:
  name: ingress-http
  namespace: default
  annotations: 
    # 指定使用的 Ingress Controller
    kubernetes.io/ingress.class: "nginx"
    # 其他常用注解
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  rules: # 路由规则
  - host: auth.example.com # 域名，相当于 nginx.conf 的 server_name
    http: # HTTP 路由规则
      paths:
      - path: / # 路径
        pathType: Prefix # 匹配类型：Prefix（前缀匹配）、Exact（精确匹配）
        backend: # 后端服务
          service:
            name: auth-service # Service 名称
            port:
              number: 3000 # Service 端口
```

**配置说明**：
- `host`：匹配的域名，支持通配符
- `pathType`：
  - `Prefix`：前缀匹配，如 `/api` 匹配 `/api/*`
  - `Exact`：精确匹配，必须完全一致
- `backend`：指定后端 Service 和端口

### Ingress 常用注解

```yaml
annotations:
  # 重写路径
  nginx.ingress.kubernetes.io/rewrite-target: /
  
  # 强制 HTTPS
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  
  # 限流（每秒请求数）
  nginx.ingress.kubernetes.io/limit-rps: "100"
  
  # 客户端最大 body 大小
  nginx.ingress.kubernetes.io/proxy-body-size: "10m"
  
  # CORS 配置
  nginx.ingress.kubernetes.io/enable-cors: "true"
  nginx.ingress.kubernetes.io/cors-allow-origin: "*"
```

## 实用技巧和最佳实践

### 1. 使用别名提高效率

在 `~/.bashrc` 或 `~/.zshrc` 中添加：

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kd='kubectl describe'
alias kl='kubectl logs'
alias ke='kubectl exec -it'
```

### 2. 使用上下文切换

```bash
# 查看所有上下文
kubectl config get-contexts

# 切换到指定上下文
kubectl config use-context ${context-name}

# 查看当前上下文
kubectl config current-context
```

### 3. 批量操作

```bash
# 删除 namespace 下所有 Pod
kubectl delete pods --all -n ${namespace}

# 删除 namespace 下所有资源
kubectl delete all --all -n ${namespace}
```

### 4. 输出格式优化

```bash
# JSON 格式输出（用于脚本处理）
kubectl get pods -o json

# 自定义列输出
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# 保存到文件
kubectl get pods -o yaml > pods.yaml
```

## 常见问题排查流程

### Pod 无法启动

```bash
# 1. 查看 Pod 状态
kubectl get pods -n ${namespace}

# 2. 查看详细描述（重点关注 Events）
kubectl describe pod ${pod-name} -n ${namespace}

# 3. 查看日志
kubectl logs ${pod-name} -n ${namespace}

# 4. 进入容器调试
kubectl exec -it ${pod-name} -n ${namespace} -- /bin/sh
```

### Service 无法访问

```bash
# 1. 检查 Service 是否存在
kubectl get svc ${service-name} -n ${namespace}

# 2. 检查 Endpoints（确认后端 Pod）
kubectl get endpoints ${service-name} -n ${namespace}

# 3. 检查 Pod 是否正常运行
kubectl get pods -l app=${app-label} -n ${namespace}

# 4. 测试 Service 连通性（在集群内 Pod 中）
kubectl run test-pod --image=busybox -it --rm -- wget -O- http://${service-name}:${port}
```

## 总结

本文整理了 Kubernetes 日常运维中最常用的命令，涵盖了 Pod、Service、Deployment 和 Ingress 四大核心资源。掌握这些命令，可以应对 80% 的日常运维场景。

**关键要点**：
1. **`describe` 命令是你的好朋友**：它提供了最详细的资源信息，包括事件和状态
2. **日志是排查问题的第一步**：大多数问题都能从日志中找到线索
3. **理解资源之间的关系**：Pod → Service → Ingress，理解这个链路很重要
4. **善用别名和上下文**：提高工作效率

希望这篇文章能帮助大家更高效地使用 Kubernetes。如果你有其他实用的命令或技巧，欢迎分享！