# WebRTC 信令：连接的"媒人"

## 前言

WebRTC 是一项强大的技术，可以实现浏览器和设备之间的点对点实时通信。但很多人在入门时都会遇到一个困惑：**WebRTC 不是说好的 P2P 吗，为什么还需要服务器？**

答案是：WebRTC 需要一个**信令服务器**来帮助两端"认识"彼此。就像两个人要通电话，首先得交换电话号码一样，WebRTC 的两端需要先交换连接信息，这个过程就是**信令（Signaling）**。

在我开发物联网设备视频直播的过程中，信令服务是最容易被忽视但又最关键的环节。本文将从实际出发，详解 WebRTC 信令的三种核心消息：**Offer、Answer 和 ICE Candidate**。

## 信令的作用：WebRTC 连接的"媒人"

### 为什么需要信令？

WebRTC 建立 P2P 连接需要解决两个问题：

1. **媒体协商**：双方需要协商使用什么编码格式、分辨率、帧率等
2. **网络穿透**：双方需要交换网络地址，找到一条能互通的路径

这两个问题的解决方案分别是：

- **SDP（Session Description Protocol）**：描述媒体能力和会话信息
- **ICE（Interactive Connectivity Establishment）**：发现和交换网络候选地址

信令服务器的职责就是**传递这些信息**，它本身不处理媒体数据，只是一个"信使"。

### 信令流程概览

```
发起方（Caller）           信令服务器           接收方（Callee）
     │                        │                     │
     │  1. 创建 Offer         │                     │
     │  ─────────────────────>│                     │
     │                        │  2. 转发 Offer      │
     │                        │────────────────────>│
     │                        │                     │ 3. 创建 Answer
     │                        │  4. 转发 Answer     │
     │                        │<────────────────────│
     │  5. 收到 Answer        │                     │
     │<───────────────────────│                     │
     │                        │                     │
     │  6. 交换 ICE Candidate（双向持续进行）        │
     │<──────────────────────>│<───────────────────>│
     │                        │                     │
     │  7. P2P 连接建立，媒体直接传输               │
     │<════════════════════════════════════════════>│
```

**关键点**：
- 信令服务器只在建立连接时使用
- 一旦 P2P 连接建立，媒体数据直接在两端传输
- 信令协议不是 WebRTC 标准的一部分，你可以用 WebSocket、HTTP、MQTT 等任何方式

## SDP Offer：发起连接请求

### 什么是 Offer？

Offer 是发起方创建的 SDP 消息，包含了发起方的媒体能力和网络信息。它相当于说："我能发送这些格式的音视频，你能接收吗？"

### 消息结构

```json
{
    "type": "offer",
    "data": {
        "deviceSn": "IPCAM123456",
        "memberId": "user789",
        "groupId": "group123",
        "sdp": "v=0\no=- 123456789...",
        "videoConfig": {
            "resolution": "1080p",
            "fps": 25,
            "bitrate": 2000000,
            "simulcast": true
        }
    }
}
```

### 字段详解

| 字段 | 说明 |
|------|------|
| `type` | 消息类型，固定为 "offer" |
| `deviceSn` | 设备序列号，用于标识设备 |
| `memberId` | 用户 ID，用于标识用户 |
| `groupId` | 群组 ID，用于多人通话场景 |
| `sdp` | SDP 描述字符串，核心内容 |
| `videoConfig` | 自定义视频配置（可选） |

### SDP 内容解析

SDP 是一个文本格式的协议，遵循 [RFC 8866](https://tools.ietf.org/html/rfc8866) 标准。一个典型的 SDP Offer 如下：

```
v=0
o=- 123456789 1 IN IP4 192.168.1.100
s=Video Stream
c=IN IP4 192.168.1.100
t=0 0
m=video 5000 RTP/AVPF 96
a=rtpmap:96 H264/90000
a=sendonly
a=ice-ufrag:abc123
a=ice-pwd:xyz789
a=fingerprint:sha-256 AA:BB:CC:DD...
```

**逐行解释**：

| 行 | 含义 |
|----|------|
| `v=0` | SDP 版本号，固定为 0 |
| `o=- 123456789 1 IN IP4 192.168.1.100` | 会话发起者信息（会话 ID、版本、IP） |
| `s=Video Stream` | 会话名称 |
| `c=IN IP4 192.168.1.100` | 连接信息（网络类型、IP 地址） |
| `t=0 0` | 时间信息（0 0 表示永久有效） |
| `m=video 5000 RTP/AVPF 96` | 媒体行：视频、端口 5000、RTP 协议、payload type 96 |
| `a=rtpmap:96 H264/90000` | 属性：payload 96 对应 H264 编码，时钟 90000Hz |
| `a=sendonly` | 属性：只发送不接收 |
| `a=ice-ufrag` / `a=ice-pwd` | ICE 认证信息 |
| `a=fingerprint` | DTLS 指纹，用于加密 |

### videoConfig 扩展字段

在实际项目中，我们通常会添加一些自定义字段来传递额外信息：

```json
"videoConfig": {
    "resolution": "1080p",    // 分辨率
    "fps": 25,                // 帧率
    "bitrate": 2000000,       // 码率（bps）
    "simulcast": true         // 是否启用 Simulcast
}
```

**Simulcast（多路编码）**：发送方同时编码多个分辨率/码率的视频流，接收方根据网络状况选择合适的流。这在多人视频会议中非常有用。

## SDP Answer：响应连接请求

### 什么是 Answer？

Answer 是接收方对 Offer 的响应，确认或调整会话参数。它相当于说："好的，我能接收这些格式，我们就用这个配置吧。"

### 消息结构

```json
{
    "type": "answer",
    "data": {
        "deviceSn": "IPCAM123456",
        "memberId": "user789",
        "groupId": "group123",
        "sdp": "v=0\no=- 987654321..."
    }
}
```

### SDP 内容示例

```
v=0
o=- 987654321 1 IN IP4 192.168.1.200
s=Video Stream
c=IN IP4 192.168.1.200
t=0 0
m=video 5002 RTP/AVPF 96
a=rtpmap:96 H264/90000
a=recvonly
a=ice-ufrag:def456
a=ice-pwd:uvw123
a=fingerprint:sha-256 EE:FF:GG:HH...
```

**关键差异**：
- `a=recvonly`：表示只接收不发送（与 Offer 的 `a=sendonly` 对应）
- 端口和 IP 是接收方的地址
- ICE 认证信息是接收方生成的

### Offer/Answer 协商过程

```
Offer (发起方)                    Answer (接收方)
─────────────────                ─────────────────
支持: H264, VP8, VP9      →      选择: H264 ✓
分辨率: 1080p, 720p       →      选择: 1080p ✓
码率: 2Mbps               →      确认: 2Mbps ✓
方向: sendonly            →      方向: recvonly ✓
```

Answer 必须是 Offer 的"子集"——只能选择 Offer 中提供的选项，不能新增。

## ICE Candidate：网络地址交换

### 什么是 ICE Candidate？

ICE Candidate 是潜在的网络连接地址（IP + 端口）。由于 NAT 和防火墙的存在，一个设备可能有多个地址：

- **host**：本地网卡地址
- **srflx（Server Reflexive）**：通过 STUN 发现的公网地址
- **relay**：通过 TURN 中继的地址

### 消息结构

```json
{
    "type": "candidate",
    "data": {
        "deviceSn": "IPCAM123456",
        "memberId": "user789",
        "groupId": "group123",
        "candidate": {
            "candidate": "candidate:1 1 UDP 2130706431 192.168.1.100 5000 typ host",
            "sdpMid": "0",
            "sdpMLineIndex": 0
        }
    }
}
```

### Candidate 字符串解析

```
candidate:1 1 UDP 2130706431 192.168.1.100 5000 typ host
          │ │  │      │           │         │       │
          │ │  │      │           │         │       └── 候选类型
          │ │  │      │           │         └── 端口
          │ │  │      │           └── IP 地址
          │ │  │      └── 优先级（数值越大越优先）
          │ │  └── 传输协议
          │ └── 组件 ID（1=RTP, 2=RTCP）
          └── 基础标识符
```

### 候选类型详解

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| `host` | 本地网卡地址 | 同一局域网内通信 |
| `srflx` | STUN 发现的公网地址 | 跨 NAT 通信（大多数情况） |
| `prflx` | 对等反射地址 | 连接过程中发现的地址 |
| `relay` | TURN 中继地址 | NAT 穿透失败时的备选 |

### ICE 候选收集过程

```javascript
// 创建 PeerConnection 时指定 ICE 服务器
const pc = new RTCPeerConnection({
    iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },        // STUN 服务器
        { urls: 'turn:turn.example.com:3478',            // TURN 服务器
          username: 'user',
          credential: 'pass' }
    ]
});

// 监听 ICE 候选事件
pc.onicecandidate = (event) => {
    if (event.candidate) {
        // 发送候选到信令服务器
        signalingServer.send({
            type: 'candidate',
            data: {
                deviceSn: 'IPCAM123456',
                candidate: event.candidate
            }
        });
    }
};
```

**收集顺序**：
1. 首先收集 `host` 候选（本地地址）
2. 然后通过 STUN 收集 `srflx` 候选
3. 如果配置了 TURN，还会收集 `relay` 候选
4. 所有候选收集完成后触发 `icegatheringstatechange` 事件

## 实际代码示例

### 发起方（Caller）

```javascript
async function startCall(deviceSn) {
    // 1. 创建 PeerConnection
    const pc = new RTCPeerConnection(iceConfig);
    
    // 2. 添加本地媒体流
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    stream.getTracks().forEach(track => pc.addTrack(track, stream));
    
    // 3. 创建 Offer
    const offer = await pc.createOffer();
    await pc.setLocalDescription(offer);
    
    // 4. 发送 Offer 到信令服务器
    signalingServer.send({
        type: 'offer',
        data: {
            deviceSn: deviceSn,
            sdp: offer.sdp,
            videoConfig: { resolution: '1080p', fps: 30 }
        }
    });
    
    // 5. 监听 ICE 候选
    pc.onicecandidate = (event) => {
        if (event.candidate) {
            signalingServer.send({
                type: 'candidate',
                data: { deviceSn, candidate: event.candidate }
            });
        }
    };
    
    return pc;
}
```

### 接收方（Callee）

```javascript
async function handleOffer(offerData) {
    // 1. 创建 PeerConnection
    const pc = new RTCPeerConnection(iceConfig);
    
    // 2. 设置远端描述（Offer）
    await pc.setRemoteDescription({
        type: 'offer',
        sdp: offerData.sdp
    });
    
    // 3. 创建 Answer
    const answer = await pc.createAnswer();
    await pc.setLocalDescription(answer);
    
    // 4. 发送 Answer 到信令服务器
    signalingServer.send({
        type: 'answer',
        data: {
            deviceSn: offerData.deviceSn,
            sdp: answer.sdp
        }
    });
    
    // 5. 监听远端媒体流
    pc.ontrack = (event) => {
        videoElement.srcObject = event.streams[0];
    };
    
    // 6. 监听 ICE 候选
    pc.onicecandidate = (event) => {
        if (event.candidate) {
            signalingServer.send({
                type: 'candidate',
                data: { deviceSn: offerData.deviceSn, candidate: event.candidate }
            });
        }
    };
    
    return pc;
}
```

### 处理远端 ICE 候选

```javascript
function handleRemoteCandidate(candidateData) {
    const candidate = new RTCIceCandidate(candidateData.candidate);
    peerConnection.addIceCandidate(candidate);
}
```

## 常见问题与调试技巧

### 1. ICE 连接失败

**症状**：`iceConnectionState` 变为 `failed`

**可能原因**：
- 双方都在对称 NAT 后面，需要 TURN 中继
- STUN/TURN 服务器配置错误
- 防火墙阻止了 UDP 流量

**调试方法**：
```javascript
pc.oniceconnectionstatechange = () => {
    console.log('ICE 状态:', pc.iceConnectionState);
};

// 查看收集到的候选
pc.onicecandidate = (e) => {
    if (e.candidate) {
        console.log('发现候选:', e.candidate.candidate);
    }
};
```

### 2. SDP 协商失败

**症状**：`setRemoteDescription` 抛出异常

**可能原因**：
- SDP 格式错误
- 不支持的编解码器
- Offer/Answer 状态不匹配

**调试方法**：
```javascript
try {
    await pc.setRemoteDescription(sdp);
} catch (error) {
    console.error('SDP 错误:', error.message);
    console.log('问题 SDP:', sdp);
}
```

### 3. 媒体流不显示

**症状**：连接成功但看不到视频

**可能原因**：
- `ontrack` 事件未正确处理
- 视频元素未设置 `autoplay`
- 媒体流方向不匹配（sendonly vs recvonly）

**调试方法**：
```javascript
pc.ontrack = (event) => {
    console.log('收到轨道:', event.track.kind);
    console.log('流 ID:', event.streams[0].id);
    
    const video = document.getElementById('remoteVideo');
    video.srcObject = event.streams[0];
    video.play().catch(e => console.error('播放失败:', e));
};
```

### 调试工具推荐

- **chrome://webrtc-internals**：Chrome 内置的 WebRTC 调试工具
- **getStats() API**：获取连接统计信息
- **Wireshark**：抓包分析 STUN/TURN 流量

## 信令服务器实现建议

### 技术选型

| 方案 | 优点 | 适用场景 |
|------|------|----------|
| WebSocket | 实时、双向、简单 | 大多数 Web 应用 |
| Socket.IO | 自动重连、房间支持 | 需要分组功能 |
| MQTT | 轻量、适合物联网 | IoT 设备 |
| HTTP 长轮询 | 兼容性好 | 受限环境 |

### 消息路由设计

```javascript
// 简单的信令服务器示例（Node.js + WebSocket）
wss.on('connection', (ws) => {
    ws.on('message', (message) => {
        const msg = JSON.parse(message);
        
        switch (msg.type) {
            case 'join':
                // 加入房间
                rooms[msg.groupId].add(ws);
                break;
            case 'offer':
            case 'answer':
            case 'candidate':
                // 转发给目标设备
                const target = findDevice(msg.data.deviceSn);
                target.send(JSON.stringify(msg));
                break;
        }
    });
});
```

## 总结

WebRTC 信令是建立 P2P 连接的关键步骤，理解 Offer、Answer 和 ICE Candidate 的作用对于开发实时通信应用至关重要。

**核心要点**：

1. **Offer/Answer 是媒体协商**：双方交换 SDP，协商编解码器、分辨率等参数
2. **ICE Candidate 是网络发现**：交换可能的网络地址，找到最佳连接路径
3. **信令服务器是信使**：只负责转发消息，不处理媒体数据
4. **状态机很重要**：按正确顺序调用 `setLocalDescription` 和 `setRemoteDescription`

**实际开发建议**：

- 先用公共 STUN 服务器（如 Google 的）测试
- 生产环境部署自己的 TURN 服务器（如 Coturn）
- 信令消息要包含足够的上下文（设备 ID、用户 ID 等）
- 做好错误处理和重连机制

---

**相关文章**：
- [P2P、SFU和MCU音视频通信架构](./P2P、SFU和MCU音视频通信架构.md)
- [RTP 实时传输协议](./RTP%20实时传输协议.md)
- [基于Luckfox Pico Max的P2P视频直播](./基于Luckfox%20Pico%20Max的P2P视频直播解决方案.md)

**参考资料**：
- [WebRTC 官方文档](https://webrtc.org/)
- [RFC 8866 - SDP](https://tools.ietf.org/html/rfc8866)
- [RFC 8445 - ICE](https://tools.ietf.org/html/rfc8445)

