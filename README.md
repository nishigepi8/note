# 技术笔记与博客文章

本仓库是我个人技术笔记和博客文章的备份，按照技术领域分类整理。

## 目录结构

```
note/
├── WebRTC/          # 音视频技术：RTP、P2P、SFU/MCU 架构
├── K8s/             # Kubernetes 容器编排与运维
├── IoT/             # 物联网：Matter 协议、MQTT 安全架构
├── DevOps/          # 开发规范：Git 分支、CI/CD
├── Architecture/    # 架构设计：授权机制、K8s 架构图
├── Projects/        # 项目实践：视频传输、嵌入式开发
├── Tools/           # 开发工具：效率提升、工具链配置
└── ABOUT.md         # 关于我
```

## 文章列表

### 🎥 WebRTC 音视频技术

| 文章 | 简介 |
|------|------|
| [P2P、SFU和MCU音视频通信架构](./WebRTC/P2P、SFU和MCU音视频通信架构.md) | 深入对比三种音视频架构的优缺点和适用场景 |
| [RTP 实时传输协议](./WebRTC/RTP%20实时传输协议.md) | RTP 协议深度解析，包括协议头、时间戳同步、性能优化 |
| [WebRTC 信令服务详解](./WebRTC/WebRTC%20信令服务详解：Offer、Answer%20与%20ICE%20Candidate.md) | Offer/Answer/ICE Candidate 详解，含代码示例和调试技巧 |
| [基于Luckfox Pico Max的P2P视频直播](./WebRTC/基于Luckfox%20Pico%20Max的P2P视频直播解决方案.md) | 低成本嵌入式 P2P 视频直播，C + Golang 混合架构 |

### ☸️ Kubernetes 容器编排

| 文章 | 简介 |
|------|------|
| [K8s常用命令](./K8s/K8s常用命令.md) | Kubernetes 日常运维命令实战指南 |
| [Kubernetes滚动更新](./K8s/Kubernetes滚动更新.md) | 滚动更新策略、配置和最佳实践 |

### 📡 物联网 IoT

| 资源 | 简介 |
|------|------|
| Matter 协议规范 | 包含 Matter 1.2 设备库规范、应用集群规范 |
| 物联网 MQTTS 安全架构 | MQTT over TLS 安全传输架构设计 |

### 🔧 DevOps 开发规范

| 文章 | 简介 |
|------|------|
| [Git 分支规则与 Commit 规范](./DevOps/水滴%20git%20分支规则.md) | 中大型团队 GitFlow 实践，Conventional Commits 规范 |

### 🏗️ 架构设计

| 文章 | 简介 |
|------|------|
| [高并发缓存同步：RSC 方案](./Architecture/高并发缓存同步：借鉴JVM%20Survivor机制的RSC方案.md) | 借鉴 JVM Survivor 机制，组合 Redis+Kafka+MongoDB 实现百万设备状态同步 |
| [GPS 轨迹存储方案深度分析](./Architecture/GPS%20轨迹存储方案深度分析：从数据结构到存储选型.md) | PostGIS、MongoDB、Redis 方案对比，含性能优化实践 |
| [自研 P2P 服务架构设计](./Architecture/自研%20P2P%20服务架构设计：从%20STUN-TURN%20到信令服务.md) | 基于 Pion 的 STUN/TURN/信令服务，支持 10 万并发 |
| Auth-Service 授权机制.pdf | 微服务授权架构设计 |
| BMGuardr Kubernetes 架构.pdf | K8s 集群架构设计方案 |
| CI 架构图.pdf | 持续集成流水线架构 |

### 💡 项目实践

| 文章 | 简介 |
|------|------|
| [Python + MQTT 实时视频传输](./Projects/使用%20Python%20和%20MQTT%20实现简单实时视频传输系统.md) | MQTT 用于视频传输的实践、性能优化与局限性分析 |

### 🔧 开发工具

| 文章 | 简介 |
|------|------|
| [Mac 开发效率提升指南](./Tools/Mac%20开发效率提升指南：从别名到工具链.md) | Alfred、iTerm2、Alias 系统，提效 10%+ |

## 技术栈

- **后端**：Java、Golang、Python
- **物联网**：Matter、MQTT、嵌入式 Linux
- **音视频**：WebRTC、RTP、H.264
- **运维**：Kubernetes、Docker、CI/CD
- **硬件**：PCB 设计、Luckfox、RV1106

## 更新日志

- 2025-12-24：重构目录结构，按技术领域分类整理
- 2025-12-21：初始迁移，优化现有文档格式和内容
- 2025-12-20：博客开源 [PowerWiki](https://github.com/steven-ld/PowerWiki.git)

## 关于

这些文章最初发布在 [ga666666.cn](https://ga666666.cn)，现在迁移到本地 Markdown 格式，便于版本控制和持续优化。

更多关于我的信息，请查看 [ABOUT.md](./ABOUT.md)。

如有问题或建议，欢迎交流讨论！
