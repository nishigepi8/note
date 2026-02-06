---
title: 企业级证书颁发系统（EJBCA）操作手册
description: 本文记录了 EJBCA 企业级证书颁发机构的完整部署和配置流程，包括 Docker 部署、管理后台配置、证书生成以及与 EMQX 的集成。涵盖证书层级架构、CRL/OCSP 吊销机制、MQTT 双向认证配置。
author: ga666666
date: 2025-01-10
updated: 2025-02-04
keywords: EJBCA, 证书颁发, PKI, TLS, 证书配置, EMQX, MQTT, CA, 证书吊销, CRL, OCSP
tags: [EJBCA, 证书, PKI, EMQX, MQTT]
---

# 企业级证书颁发系统（EJBCA）操作手册

## 什么是 EJBCA？

EJBCA（Enterprise Java Beans Certificate Authority）是一个**企业级公钥基础设施（PKI）软件**，用于颁发、管理和验证数字证书。

### 通俗解释

你可以把 EJBCA 想象成一个"数字证书工厂"：

- **颁发证书**：像工厂生产产品一样，EJBCA 可以为服务器、设备、用户颁发数字证书
- **管理证书**：像仓库管理一样，EJBCA 可以追踪所有证书的状态（有效、过期、吊销）
- **验证证书**：像一个验证中心，EJBCA 可以确认证书是否真实有效

### 常见应用场景

| 场景 | 说明 |
|------|------|
| HTTPS 网站 | 为 Web 服务器颁发 TLS 证书，实现加密访问 |
| MQTT 双向认证 | 为 IoT 设备颁发客户端证书，实现设备身份验证 |
| 代码签名 | 为软件颁发签名证书，确保代码来源可信 |
| 邮件加密 | 为邮件证书颁发 S/MIME 证书，实现邮件加密 |

### 为什么选择 EJBCA？

- **开源免费**：Community Edition 免费使用
- **功能完整**：支持证书颁发、吊销、CRL/OCSP 等完整 PKI 功能
- **企业级**：经过大规模生产环境验证
- **标准兼容**：符合 X.509、PKCS 标准

## 前言

EJBCA（Enterprise Java Beans Certificate Authority）是一个功能强大的企业级证书颁发机构系统，本文记录了在生产环境中部署和配置 EJBCA 的完整流程，涵盖 Docker 部署、管理后台配置、证书生成以及与 EMQX 的集成。

**适用范围**：
- 企业内部 PKI 体系搭建
- MQTT 双向认证证书管理
- IoT 设备证书颁发与吊销

---

## 一、EJBCA 证书平台部署

### 1.1 前置条件

- JDK 17+
- MySQL 5.7+ / MariaDB 10.5+
- Docker 环境

> **官方文档**：[EJBCA Installation Guide](https://docs.keyfactor.com/ejbca/latest/ejbca-installation)

### 1.2 Docker 部署

#### 快速启动（测试环境）

```bash
# 使用内置 H2 数据库快速验证
docker run -it --rm \
  -p 80:8080 \
  -p 443:8443 \
  -h localhost \
  -e TLS_SETUP_ENABLED="simple" \
  keyfactor/ejbca-ce
```

#### 生产环境部署

```bash
docker run -it -d \
  --name=ejbca-9.0.0 \
  -p 9080:8080 \
  -p 9443:8443 \
  -h dev-ejbca.dl-aiot.com \
  --restart on-failure:3 \
  -e TLS_SETUP_ENABLED="true" \
  -e DATABASE_JDBC_URL="jdbc:mysql://192.168.10.106:3306/dl_ejbca" \
  -e DATABASE_USER="root" \
  -e DATABASE_PASSWORD="your_password" \
  keyfactor/ejbca-ce:9.0.0
```

![Docker 启动成功](./images/ejbca-guide-14.png)

### 1.3 获取管理后台访问凭证

首次启动成功后，控制台会显示访问地址和初始密码：

```yaml
URL:      https://localhost:443/ejbca/ra/enrollwithusername.xhtml?username=superadmin
Password: BSQ9Ku3btOadV8s+pLVxJ/Ug
```

> **注意**：首次访问建议使用无痕窗口，避免浏览器缓存导致的问题。

---

## 二、配置 EJBCA 管理后台

### 2.1 初次访问配置

使用 Firefox 浏览器访问管理后台，输入初始密码登录。

![登录页面](./images/ejbca-guide-13.png)

按照向导完成初始配置：

1. **接受证书**：浏览器会提示证书不受信任，继续访问即可
2. **创建超级管理员**：设置 superadmin 用户密码
3. **初始化数据库**：配置证书存储后端

![管理员设置](./images/ejbca-guide-01.png)
![管理员配置](./images/ejbca-guide-02.png)
![最终确认](./images/ejbca-guide-03.png)

> **提示**：建议新建隐私窗口进行初始配置，避免缓存问题。

![配置完成](./images/ejbca-guide-04.png)

### 2.2 后续访问方式

配置完成后，后续访问需要使用管理员证书：

1. 从内部 Wiki 获取超级管理员证书
2. 导入 SuperAdmin.p12 到浏览器

![证书下载页面](./images/ejbca-guide-06.png)

---

## 三、证书接入方案对比

### 方案对比

| 方案 | 优点 | 缺点 | 推荐度 |
|------|------|------|--------|
| 管理后台手动生成 | 简单直观 | 无法自动化 | ★☆☆ |
| ejbca-easy-rest-client | 支持命令行 | 需二次开发 | ★★★ |
| REST API 自行集成 | 灵活性高 | 开发成本高 | ★★★★ |

### 3.1 管理后台生成证书

适用于证书数量少、生成频率低的场景。

![证书申请页面](./images/ejbca-guide-11.png)
![填写证书信息](./images/ejbca-guide-12.png)
![生成证书](./images/ejbca-guide-15.png)
![证书下载](./images/ejbca-guide-16.png)

### 3.2 ejbca-easy-rest-client

**项目地址**：[Keyfactor/ejbca-easy-rest-client](https://github.com/Keyfactor/ejbca-easy-rest-client)

#### 安装与使用

```bash
# 进入 target 目录执行
java -jar target/erce-0.3.0-SNAPSHOT.jar enroll genkeys \
  --authkeystore /path/to/SuperAdmin.p12 \
  --authkeystorepass your_password \
  --endentityprofile "ClientAuth" \
  --certificateprofile "ClientAuth" \
  --ca ManagementCA \
  --subjectaltname "dnsName=dev-mqtt.dl-aiot.com" \
  --hostname localhost \
  --destination ./certs \
  --subjectdn "CN=test-device-01" \
  --username test-device-01 \
  --keyalg EC --keyspec P-256 --verbose
```

![命令执行](./images/ejbca-guide-17.png)
![证书生成结果](./images/ejbca-guide-18.png)
![证书列表](./images/ejbca-guide-19.png)

#### 证书提取

```bash
# 提取证书
openssl pkcs12 -in certificate.pfx -nokeys -out certificate.pem

# 提取私钥
openssl pkcs12 -in certificate.pfx -nocerts -out key.pem

# 移除私钥密码
openssl rsa -in key.pem -out key_nopass.pem
```

![证书提取](./images/ejbca-guide-20.png)
![证书信息](./images/ejbca-guide-21.png)
![私钥处理](./images/ejbca-guide-22.png)
![完成](./images/ejbca-guide-23.png)

> **注意**：此方案需要二次开发才能集成到生产系统，工作量较大。

### 3.3 REST API 自行集成

推荐用于需要自动化证书管理的生产环境。

**适用场景**：
- 设备证书批量颁发
- 证书到期自动续期
- 证书吊销实时同步

---

## 四、CA 证书体系配置

### 4.1 证书层级架构

```mermaid
graph TB
    subgraph "根证书层级"
        RootCA["DesignXXXXXCA<br/>(Root CA)"]
    end

    subgraph "中间证书层级"
        IntCA["PetXXXXCA<br/>(Intermediate CA)"]
    end

    subgraph "终端证书"
        ServerCert["EMQX Server Cert<br/>(Server Auth)"]
        ClientCert["User Client Cert<br/>(Client Auth)"]
    end

    RootCA -->|"签名"| IntCA
    IntCA -->|"签名"| ServerCert
    IntCA -->|"签名"| ClientCert
```

### 4.2 创建根证书

![创建根 CA](./images/ejbca-guide-27.png)
![配置根 CA](./images/ejbca-guide-28.png)
![密钥生成](./images/ejbca-guide-29.png)
![证书签名](./images/ejbca-guide-30.png)
![证书颁发](./images/ejbca-guide-31.png)
![确认页面](./images/ejbca-guide-32.png)
![完成](./images/ejbca-guide-33.png)

### 4.3 创建中间证书

中间证书用于隔离不同业务线的证书管理权限。

![中间 CA 列表](./images/ejbca-guide-34.png)
![创建中间 CA](./images/ejbca-guide-35.png)
![选择父 CA](./images/ejbca-guide-36.png)
![配置信息](./images/ejbca-guide-37.png)
![完成配置](./images/ejbca-guide-38.png)
![证书链验证](./images/ejbca-guide-39.png)
![最终确认](./images/ejbca-guide-40.png)

### 4.4 证书提取命令速查

```bash
# 提取证书
openssl pkcs12 -in demo-api.dl-aiot.com.p12 -nokeys -out certificate.pem

# 提取私钥
openssl pkcs12 -in demo-api.dl-aiot.com.p12 -nocerts -out key.pem

# 解密私钥（移除密码保护）
openssl rsa -in key.pem -out key_nopass.pem
```

---

## 五、证书吊销机制

### 5.1 CRL（证书吊销列表）

#### 配置 CRL 发布点

![CRL 配置入口](./images/ejbca-guide-52.png)
![新建 CRL 发布点](./images/ejbca-guide-53.png)
![CRL 验证](./images/ejbca-guide-54.png)

#### 创建吊销规则

![创建吊销配置](./images/ejbca-guide-55.png)

#### 证书验证命令

```bash
openssl ocsp \
  -issuer ManagementCA.pem \
  -cert user-cert.pem \
  -url https://ejbca.example.com:8443/ejbca/publicweb/status/ocsp \
  -CAfile ManagementCA.pem
```

### 5.2 OCSP（在线证书状态协议）

> OCSP 提供实时证书状态查询，比 CRL 更适合高并发场景。

![OCSP 配置入口](./images/ejbca-guide-57.png)
![OCSP 响应器](./images/ejbca-guide-58.png)
![状态查询](./images/ejbca-guide-59.png)

---

## 六、EMQX 证书集成

### 6.1 TLS 证书配置

**EMQX 版本**：4.4.19

![TLS 配置入口](./images/ejbca-guide-56.png)

#### 配置步骤

1. 上传根证书（Root CA）
2. 上传中间证书（如有）
3. 上传服务器证书
4. 配置私钥

### 6.2 OCSP Stapling 配置

![OCSP 配置](./images/ejbca-guide-60.png)
![OCSP 响应](./images/ejbca-guide-61.png)
![验证配置](./images/ejbca-guide-62.png)
![完成](./images/ejbca-guide-63.png)

### 6.3 CRL 配置

**EMQX 版本**：5.8.4

![CRL 配置](./images/ejbca-guide-64.png)

#### 验证吊销功能

使用已吊销的证书尝试连接 EMQX，连接应被拒绝。

---

## 七、最佳实践

### 7.1 证书生命周期管理

| 阶段 | 操作 | 建议周期 |
|------|------|----------|
| 根证书 | 创建后长期使用 | 10-20 年 |
| 中间证书 | 每年轮换 | 1 年 |
| 服务器证书 | 每年更新 | 1 年 |
| 客户端证书 | 根据业务需求 | 1-3 年 |

### 7.2 安全建议

1. **根证书离线存储**：根证书私钥应离线保存，仅在签发中间证书时使用
2. **最小权限原则**：不同业务使用不同的中间证书
3. **定期轮换**：定期轮换证书和密钥
4. **吊销机制**：及时吊销已泄露或不再使用的证书

---

**参考资源**：
- [EJBCA 官方文档](https://docs.keyfactor.com/ejbca/latest/)
- [ejbca-easy-rest-client](https://github.com/Keyfactor/ejbca-easy-rest-client)
- [EMQX SSL/TLS 文档](https://www.emqx.io/docs/en/v4.4/advanced/ssl.html)
