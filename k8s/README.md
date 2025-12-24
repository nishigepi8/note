# Kubernetes 容器编排

本目录包含 Kubernetes 运维和开发相关的实战文章。

## 文章列表

| 文章 | 关键词 | 场景 |
|------|--------|------|
| [Kubernetes 常用命令实战指南](/post/K8s/Kubernetes%20常用命令实战指南.md) | kubectl、Pod、Service、调试 | 日常运维 |
| [Kubernetes 滚动更新实战指南](/post/K8s/Kubernetes%20滚动更新实战指南.md) | Deployment、策略、回滚、健康检查 | 发布上线 |

## 核心概念

- **Pod**：最小部署单元，包含一个或多个容器
- **Service**：为 Pod 提供稳定的网络访问入口
- **Deployment**：管理 Pod 的副本和更新策略
- **Ingress**：提供 HTTP/HTTPS 路由规则
- **ConfigMap/Secret**：配置和敏感信息管理

## 学习路径

1. 熟悉 kubectl 常用命令，能快速排查问题
2. 理解 Deployment 滚动更新机制
3. 掌握服务发现和负载均衡（Service、Ingress）
4. 深入理解资源调度和自动伸缩

## 推荐资源

- [Kubernetes 官方文档](https://kubernetes.io/zh-cn/docs/)
- [Kubernetes Patterns](https://www.oreilly.com/library/view/kubernetes-patterns/9781492050278/)

