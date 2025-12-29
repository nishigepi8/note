# 零停机更新策略

## 前言

在生产环境中，应用的更新是不可避免的。Kubernetes 的滚动更新（Rolling Update）机制让我们可以在不中断服务的情况下，平滑地将应用从旧版本升级到新版本。本文将深入解析滚动更新的原理、配置策略和最佳实践，帮助你在实际工作中更好地使用这一功能。

## 滚动更新原理

Deployment 对象可以定义一个副本集（ReplicaSet），并且支持滚动更新。具体来说，滚动更新会先在新的 ReplicaSet 中启动一些 Pod，然后逐步停止旧的 ReplicaSet 中的 Pod，直到所有的 Pod 都被更新完成。

### 更新流程

```
旧版本 Pod (v1)         新版本 Pod (v2)
    ↓                        ↓
[Pod1, Pod2, Pod3]  →  [Pod1, Pod2, Pod3, Pod4(v2)]
    ↓                        ↓
[Pod1, Pod2, Pod4(v2)] → [Pod1, Pod4(v2), Pod5(v2)]
    ↓                        ↓
[Pod4(v2), Pod5(v2), Pod6(v2)]  ← 全部更新完成
```

**关键点**：
- 新旧 Pod 同时运行，保证服务不中断
- 逐步替换，可以控制更新速度
- 支持回滚，如果新版本有问题可以快速恢复

## 配置示例

以下是一个完整的 Deployment 配置文件：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
        ports:
        - containerPort: 80
        # 健康检查，确保新 Pod 就绪后再继续更新
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # 最多不可用的 Pod 数量
      maxSurge: 1        # 最多可以超出期望副本数的 Pod 数量
```

### 参数说明

- **spec.replicas**：指定要启动的 Pod 的数量
- **spec.selector**：指定要选择的 Pod 的标签
- **spec.template**：定义要创建的 Pod 的配置
- **spec.strategy.type**：更新策略类型，`RollingUpdate` 或 `Recreate`
- **spec.strategy.rollingUpdate.maxUnavailable**：在滚动更新期间最多可以停止的 Pod 数量
- **spec.strategy.rollingUpdate.maxSurge**：在滚动更新期间最多可以启动的 Pod 数量

## 更新策略详解

### maxUnavailable 和 maxSurge 的组合

这两个参数共同控制更新速度，理解它们的关系很重要：

| maxUnavailable | maxSurge | 更新行为 | 适用场景 |
|----------------|----------|----------|----------|
| 1 | 1 | 先启动 1 个新 Pod，再停止 1 个旧 Pod | **推荐**：平衡速度和稳定性 |
| 0 | 1 | 先启动 1 个新 Pod，等就绪后再停止旧 Pod | 最保守：保证始终有足够 Pod |
| 1 | 0 | 先停止 1 个旧 Pod，再启动 1 个新 Pod | 资源受限：不能超出副本数 |
| 25% | 25% | 按百分比控制 | 大规模部署：自动适应副本数变化 |

**我的经验**：
- 小规模应用（3-5 个副本）：`maxUnavailable: 1, maxSurge: 1`
- 中规模应用（10+ 副本）：`maxUnavailable: 25%, maxSurge: 25%`
- 关键服务：`maxUnavailable: 0, maxSurge: 1`，确保零停机

### Recreate 策略

```yaml
strategy:
  type: Recreate
```

**特点**：
- 先停止所有旧 Pod，再启动新 Pod
- 会有短暂的服务中断
- 适合有状态应用，需要确保只有一个版本运行

## 执行滚动更新

### 方法 1：更新镜像版本

```bash
# 直接更新镜像
kubectl set image deployment/myapp-deployment myapp=myapp:v2

# 查看更新状态
kubectl rollout status deployment/myapp-deployment

# 查看更新历史
kubectl rollout history deployment/myapp-deployment
```

### 方法 2：修改 YAML 文件

```bash
# 编辑 Deployment
kubectl edit deployment/myapp-deployment

# 或修改文件后应用
kubectl apply -f deployment.yaml
```

### 方法 3：使用 kubectl patch

```bash
kubectl patch deployment/myapp-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"myapp","image":"myapp:v2"}]}}}}'
```

## 监控更新过程

### 实时查看更新状态

```bash
# 查看 Deployment 状态
kubectl get deployment myapp-deployment -w

# 查看 Pod 状态
kubectl get pods -l app=myapp -w

# 查看 ReplicaSet 变化
kubectl get rs -w
```

### 更新过程中的 Pod 状态

```
NAME                                READY   STATUS
myapp-deployment-7d4b8c9f5d-xxx     1/1     Running    # 旧版本
myapp-deployment-7d4b8c9f5d-yyy     1/1     Running    # 旧版本
myapp-deployment-7d4b8c9f5d-zzz     1/1     Running    # 旧版本
myapp-deployment-8a5c9d0e6f-aaa      0/1     ContainerCreating  # 新版本启动中
myapp-deployment-8a5c9d0e6f-aaa      1/1     Running    # 新版本就绪
myapp-deployment-7d4b8c9f5d-xxx     1/1     Terminating  # 旧版本停止中
```

## 回滚操作

### 快速回滚到上一版本

```bash
# 回滚到上一个版本
kubectl rollout undo deployment/myapp-deployment

# 查看回滚状态
kubectl rollout status deployment/myapp-deployment
```

### 回滚到指定版本

```bash
# 查看历史版本
kubectl rollout history deployment/myapp-deployment

# 回滚到指定版本（例如 revision 2）
kubectl rollout undo deployment/myapp-deployment --to-revision=2
```

### 记录更新原因

在更新时添加注释，方便后续查看：

```bash
kubectl annotate deployment/myapp-deployment \
  kubernetes.io/change-cause="升级到 v2，修复内存泄漏问题"
```

## 最佳实践

### 1. 健康检查是必须的

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
  successThreshold: 1
  failureThreshold: 3
```

**重要性**：没有健康检查，Kubernetes 无法判断新 Pod 是否就绪，可能导致流量打到未就绪的 Pod。

### 2. 优雅关闭（Graceful Shutdown）

```yaml
spec:
  containers:
  - name: myapp
    lifecycle:
      preStop:
        exec:
          command: ["/bin/sh", "-c", "sleep 15"]  # 给时间处理完现有请求
  terminationGracePeriodSeconds: 30  # 默认 30 秒
```

**原理**：Pod 停止前会先执行 `preStop` 钩子，然后等待 `terminationGracePeriodSeconds` 时间，让应用处理完现有请求。

### 3. 资源限制

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

**好处**：避免新 Pod 启动时资源竞争，导致旧 Pod 被 OOMKilled。

### 4. 分阶段更新

对于大规模部署，可以分阶段更新：

```bash
# 第一阶段：更新 10% 的 Pod
kubectl set image deployment/myapp-deployment myapp=myapp:v2
kubectl scale deployment/myapp-deployment --replicas=3  # 先小规模测试

# 观察一段时间，确认无问题后
# 第二阶段：全量更新
kubectl scale deployment/myapp-deployment --replicas=10
```

## 常见问题排查

### 1. 更新卡住不动

**可能原因**：
- 新镜像拉取失败
- 新 Pod 健康检查不通过
- 资源不足

**排查方法**：
```bash
# 查看 Pod 事件
kubectl describe pod <pod-name>

# 查看 Pod 日志
kubectl logs <pod-name>

# 查看 Deployment 事件
kubectl describe deployment/myapp-deployment
```

### 2. 更新后服务异常

**快速回滚**：
```bash
kubectl rollout undo deployment/myapp-deployment
```

**分析原因**：
- 检查新版本配置是否正确
- 检查环境变量、ConfigMap、Secret 是否变化
- 检查依赖服务是否兼容

### 3. 更新速度太慢

**优化方法**：
- 增加 `maxSurge` 和 `maxUnavailable`
- 优化镜像大小，加快拉取速度
- 使用本地镜像仓库
- 优化健康检查时间（减少 `initialDelaySeconds`）

## 总结

Kubernetes 的滚动更新是一个强大且灵活的功能，正确使用可以大大提升部署的可靠性和效率。关键要点：

1. **合理配置策略**：根据应用特点选择 `maxUnavailable` 和 `maxSurge`
2. **健康检查必须**：确保新 Pod 就绪后再接收流量
3. **优雅关闭**：给应用时间处理完现有请求
4. **监控和回滚**：实时监控更新过程，准备好回滚方案

希望这篇文章能帮助你在生产环境中更好地使用滚动更新功能。如有问题，欢迎交流讨论！