# 音视频技术

本目录包含 WebRTC 相关的技术文章，涵盖协议原理、架构设计和实践经验。

## 文章列表

| 文章 | 关键词 | 难度 |
|------|--------|------|
| [P2P、SFU和MCU音视频通信架构](./P2P、SFU和MCU音视频通信架构.md) | 架构对比、扩展性、成本分析 | ⭐⭐⭐ |
| [RTP 实时传输协议](./RTP%20实时传输协议.md) | 协议头、时间戳、序列号、RTCP | ⭐⭐⭐⭐ |
| [WebRTC 信令服务详解](./WebRTC%20信令服务详解：Offer、Answer%20与%20ICE%20Candidate.md) | SDP、ICE、Offer/Answer、信令 | ⭐⭐⭐ |
| [基于Luckfox Pico Max的P2P视频直播](./基于Luckfox%20Pico%20Max的P2P视频直播解决方案.md) | 嵌入式、H.264、NAT穿透、Pion | ⭐⭐⭐⭐ |

## 技术栈

- **协议**：RTP/RTCP、ICE、STUN/TURN、SDP
- **编码**：H.264、VP8/VP9、Opus
- **库**：Pion WebRTC、libwebrtc、GStreamer
- **硬件**：Luckfox Pico Max、RV1106

## 学习路径

1. 先理解 P2P/SFU/MCU 架构差异和适用场景
2. 深入学习 RTP 协议细节，理解时间戳和同步
3. 通过实际项目（如 Luckfox P2P 直播）巩固知识

## 推荐资源

- [WebRTC for the Curious](https://webrtcforthecurious.com/)
- [Pion WebRTC](https://github.com/pion/webrtc)
- [High Performance Browser Networking - WebRTC](https://hpbn.co/webrtc/)

