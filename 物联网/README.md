# 物联网 IoT

本目录包含物联网相关的技术资料，重点是 Matter 协议和 MQTT 安全架构。

## 资源列表

### Matter 协议

| 文件 | 说明 |
|------|------|
| Matter Introduction.pdf | Matter 协议入门介绍 |
| Matter 深圳物联网展.pdf | 展会资料，行业趋势 |
| Matter-1.2-Application-Cluster-Specification.pdf | 应用集群规范 |
| Matter-1.2-Device-Library-Specification.pdf | 设备库规范 |
| Application-Clusters-Door-Lock.pdf | 门锁应用集群详解 |

### MQTT 安全

| 文件 | 说明 |
|------|------|
| 物联网_MQTTS_安全性能架构.pdf | MQTT over TLS 安全传输架构设计 |

## Matter 协议简介

Matter 是由 CSA（连接标准联盟）制定的智能家居统一标准，旨在解决智能家居设备互联互通的问题。

**核心特点**：
- **多协议支持**：可运行在 Thread、Wi-Fi、以太网之上
- **本地化控制**：无需云服务即可实现设备间通信
- **安全性**：端到端加密，区块链证书链
- **互操作性**：不同厂商设备可以直接通信

## 技术栈

- **协议**：Matter、MQTT、CoAP、Thread
- **安全**：TLS、DTLS、区块链证书
- **硬件**：ESP32、nRF52、RV1106

## 推荐资源

- [Matter 官方文档](https://csa-iot.org/all-solutions/matter/)
- [EMQX MQTT Broker](https://www.emqx.io/)

