---
title: 音视频技术
description: 本目录包含 WebRTC 相关的技术文章，涵盖协议原理、架构设计和实践经验。
author: ga666666
date: 2026-01-10
updated: 2026-01-10
keywords: 音视频技术, 文章列表, 技术栈, 学习路径, 推荐资源, 协议, 编码, 硬件
tags: [音视频, WebRTC]
---


# 音视频技术

本目录包含 WebRTC 相关的技术文章，涵盖协议原理、架构设计和实践经验。

## 快速导航

### 📚 学习路径

**第一阶段：架构理解**
- [P2P、SFU和MCU音视频通信架构](./P2P、SFU和MCU音视频通信架构.md) ⭐⭐⭐
  - 理解三种架构的差异和适用场景
  - 成本、延迟、扩展性对比

**第二阶段：协议深入**
- [RTP 实时传输协议](./RTP 实时传输协议.md) ⭐⭐⭐⭐
  - 时间戳、序列号、RTCP 反馈
  - 实时性和可靠性的权衡

**第三阶段：信令与连接**
- [WebRTC 信令服务详解](./WebRTC 信令服务详解.md) ⭐⭐⭐
  - SDP、ICE、Offer/Answer 模型
  - NAT 穿透与 STUN/TURN

**第四阶段：实战项目**
- [Luckfox Pico Max P2P直播方案](./Luckfox Pico Max P2P直播方案.md) ⭐⭐⭐⭐
  - 嵌入式设备上的 WebRTC 实现
  - H.264 编码与 Pion 库使用

## 文章列表

| 文章 | 关键词 | 难度 |
|------|--------|------|
| [P2P、SFU和MCU音视频通信架构](./P2P、SFU和MCU音视频通信架构.md) | 架构对比、扩展性、成本分析 | ⭐⭐⭐ |
| [RTP 实时传输协议](./RTP 实时传输协议.md) | 协议头、时间戳、序列号、RTCP | ⭐⭐⭐⭐ |
| [WebRTC 信令服务详解](./WebRTC 信令服务详解.md) | SDP、ICE、Offer/Answer、信令 | ⭐⭐⭐ |
| [Luckfox Pico Max P2P直播方案](./Luckfox Pico Max P2P直播方案.md) | 嵌入式、H.264、NAT穿透、Pion | ⭐⭐⭐⭐ |

## 技术栈

- **协议**：RTP/RTCP、ICE、STUN/TURN、SDP
- **编码**：H.264、VP8/VP9、Opus
- **库**：Pion WebRTC、libwebrtc、GStreamer
- **硬件**：Luckfox Pico Max、RV1106

## 学习路径

1. 先理解 P2P/SFU/MCU 架构差异和适用场景
2. 深入学习 RTP 协议细节，理解时间戳和同步
3. 学习 WebRTC 信令和 NAT 穿透技术
4. 通过实际项目（如 Luckfox P2P 直播）巩固知识

## 推荐资源

- [WebRTC for the Curious](https://webrtcforthecurious.com/)
- [Pion WebRTC](https://github.com/pion/webrtc)
- [High Performance Browser Networking - WebRTC](https://hpbn.co/webrtc/)
