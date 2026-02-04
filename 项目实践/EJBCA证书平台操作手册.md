> 参考文档：https://blog.csdn.net/u011391839/article/details/89471514

# 一、EJBCA证书平台部署

官方文档：https://docs.keyfactor.com/ejbca/latest/ejbca-installation

> 前置条件包含 JDK17、MySQL
>
> Docker 镜像中已包含，验证阶段使用内置 H2 [MariaDB](https://downloads.mariadb.org/)

## 1.1 运行 docker 镜像

```sas
docker run -it --rm -p 80:8080 -p 443:8443 -h localhost -e TLS_SETUP_ENABLED="simple" keyfactor/ejbca-ce


docker run -it -d --name=ejbca-9.0.0 \
  -p 9080:8080 -p 9443:8443 \
  -h dev-ejbca.dl-aiot.com \
  --restart on-failure:3 \
  -e TLS_SETUP_ENABLED="true" \
  -e DATABASE_JDBC_URL="jdbc:mysql://192.168.10.106:3306/dl_ejbca" \
  -e DATABASE_USER="root" \
  -e DATABASE_PASSWORD="HhB7WwxbWCNMxfJhkbqc" \
  keyfactor/ejbca-ce:9.0.0
  
  
docker run -it -d --name=ejbca   -p 9080:8080 -p 9443:8443   -h 10.51.64.10  --restart on-failure:3   -e TLS_SETUP_ENABLED="true"   -e DATABASE_JDBC_URL="jdbc:mysql://10.50.128.14:3306/dl_ejbca"   -e DATABASE_USER="root"   -e DATABASE_PASSWORD="HhB7WwxbWCNMxfJhkbqc"   keyfactor/ejbca-ce:9.0.0
```

![](images/EJBCA证书平台操作手册-image-14.png)

## 1.2 获取管理后台连接 & 密码

第一次启动成功会显示管理后台连接&密码

```yaml
URL:      https://localhost:443/ejbca/ra/enrollwithusername.xhtml?username=superadmin
Password: BSQ9Ku3btOadV8s+pLVxJ/Ug
```

# 二、配置 EJBCA 管理后台

## 2.1 初次访问管理后台

使用火狐浏览器访问, 输入上一步获取的密码

![](images/EJBCA证书平台操作手册-image-13.png)

![](images/EJBCA证书平台操作手册-image.png)

![](images/EJBCA证书平台操作手册-image-1.png)

![](images/EJBCA证书平台操作手册-image-2.png)

![](images/EJBCA证书平台操作手册-image-3.png)

**<span style="color: inherit; background-color: rgba(255,246,122,0.8)">新建隐私窗口</span>，避免缓存**

![](images/EJBCA证书平台操作手册-image-4.png)

成功进入后台管理页面

![](images/EJBCA证书平台操作手册-image-5.png)

## 2.2 后续访问

[ DL-wiki](https://i7uduvdvau.feishu.cn/wiki/wikcnHE86PXC0zmUuqFeuWpWsMf)中有相关证书以及密码

![](images/EJBCA证书平台操作手册-image-6.png)

下载SuperAdmin.p12


![](images/EJBCA证书平台操作手册-image-7.png)

![](images/EJBCA证书平台操作手册-image-8.png)

![](images/EJBCA证书平台操作手册-image-9.png)

新建隐私窗口，避免缓存

![](images/EJBCA证书平台操作手册-image-10.png)

# 三、云平台接入 EJBCA 平台

## 3.1 基于ejbca-easy-rest-client

**流程**：

1. 通过管理后台或命令行快速生成证书。

2. 支持多种选项，如证书绑定域名、设置密码等。

3. 利用 Java JAR 文件执行命令行生成证书。

**优点**：

1. 验证可行性高，文档清晰。

2. 提供灵活的参数配置，支持大多数场景。

3. 可利用 OpenSSL 提取证书和密钥，便于整合到其他服务。

**缺点**：

1. 手动命令行操作，无法与现有服务自动化集成。

2. 需要额外的脚本化或 SDK 修改工作量。



<span style="color: inherit; background-color: rgb(255,233,40)">此方式已验证可行，要集成到服务端，需要修改</span> <span style="color: inherit; background-color: rgb(255,233,40)">源码</span> <span style="color: inherit; background-color: rgb(255,233,40)">，工作量较大</span>

### 3.1.1管理后台生成

![](images/EJBCA证书平台操作手册-image-11.png)

![](images/EJBCA证书平台操作手册-image-12.png)

![](images/EJBCA证书平台操作手册-image-15.png)

![](images/EJBCA证书平台操作手册-image-16.png)

### 3.1.2 Java-SDK

> https://github.com/Keyfactor/ejbca-easy-rest-client

![](images/EJBCA证书平台操作手册-image-17.png)

进入 target，执行指令

```scala
# 执行后输入密码
java -jar target/erce-0.3.0-SNAPSHOT.jar enroll genkeys \
--authkeystore /Users/ga666666/Desktop/SuperAdmin.p12 \
--authkeystorepass 54lE9avyVVlut72lUunWQV8m \
--endentityprofile "ClientAuth" \
--certificateprofile "ClientAuth" \
--ca ManagementCA \
--subjectaltname "dnsName=dev-mqtt.dl-aiot.com" \
--hostname localhost \
--destination ./certs \
--subjectdn "CN=test-erces-01.test" \
--username test-erces-01.test -p \
--keyalg EC --keyspec P-256 --verbose
```

![](images/EJBCA证书平台操作手册-image-18.png)

![](images/EJBCA证书平台操作手册-image-19.png)





## ~~3.2 基于ejbca-java-client-sdk~~

* **分析**：

  1. 官方 SDK 提供 Java 方法封装，理论上适合与后端系统直接对接。

  2. 实测无法正常使用，长期无人维护。

  3. 文档和社区支持较少，无法定位问题。

* **建议**：无法使用 SDK，后续可跟踪更新版本。



```shell
docker run -it -d --name=ejbca  -p 80:8080 -p 443:8443 -p 8080:8080 -p 8443:8443  -h localhost -e TLS_SETUP_ENABLED="true" keyfactor/ejbca-ce
```

![](images/EJBCA证书平台操作手册-image-20.png)

![](images/EJBCA证书平台操作手册-image-21.png)

![](images/EJBCA证书平台操作手册-image-22.png)

**提取证书**

```sql
openssl pkcs12 -in certificate.pfx -nokeys -out certificate.pem
```

**提取密钥**

```sql
openssl pkcs12 -in certificate.pfx -nocerts -out key.pem
```

* `-nokeys`: 不包含密钥。

* `-nocerts`: 不包含证书。

* `-out`: 输出的 PEM 文件。

![](images/EJBCA证书平台操作手册-image-23.png)

移除密码保护：

```sql
openssl rsa -in key.pem -out key_nopass.pem
```

此时无需再输入 PEM 密码，导出的 `key_nopass.pem` 即为不带密码的私钥。

<span style="color: inherit; background-color: rgb(255,233,40)">但是进行连接仍然提示没有鉴权的错误</span>



## 3.3 基于 Rest-API 自行集成

> 初步验证可行，需要实现相关的接口

**流程**：

1. 利用 REST API 提供的接口进行证书管理。

2. 可实现证书生成、查询、更新等全流程管理。

**优点**：

1. 灵活性高，便于与现有服务集成。

2. 提供标准化 HTTP 接口，易于调试和扩展。

3. 支持完整的认证和权限管理流程。

**缺点**：

1. 需要开发 REST 客户端接口，初期开发成本较高。

2. 文档覆盖不完整，部分细节需要结合实际环境调试。

![](images/EJBCA证书平台操作手册-image-24.png)



![](images/EJBCA证书平台操作手册-image-25.png)

![](images/EJBCA证书平台操作手册-image-26.png)

**综合分析**

# 四、EJBCA CA配置

## 4.1 基于根CA生成证书

![](images/EJBCA证书平台操作手册-image-27.png)

![](images/EJBCA证书平台操作手册-image-28.png)

![](images/EJBCA证书平台操作手册-image-29.png)

![](images/EJBCA证书平台操作手册-image-30.png)

![](images/EJBCA证书平台操作手册-image-31.png)

![](images/EJBCA证书平台操作手册-image-32.png)

![](images/EJBCA证书平台操作手册-image-33.png)

![](images/EJBCA证书平台操作手册-image-34.png)

![](images/EJBCA证书平台操作手册-image-35.png)

![](images/EJBCA证书平台操作手册-image-36.png)

![](images/EJBCA证书平台操作手册-image-37.png)

![](images/EJBCA证书平台操作手册-image-38.png)

![](images/EJBCA证书平台操作手册-image-39.png)

![](images/EJBCA证书平台操作手册-image-40.png)

**P12 证书解析**

```shell
// 提取证书
openssl pkcs12 -in demo-api.dl-aiot.com.p12 -nokeys -out certificate.pem
// 提取密码
openssl pkcs12 -in demo-api.dl-aiot.com.p12 -nocerts -out key.pem
// 解密证书的KEY
openssl rsa -in key.pem -out key_nopass.pem
```

## 4.2 基于根CA配置二级CA生成证书

```sql
+--------------------+
|  DesignLibroCA    |
|  (Root CA)        |
+--------------------+
          |
          |
          v
+--------------------+
|  PetlibroCA       |
|  (Intermediate CA)|
+--------------------+
          |
          +-----------------+-----------------+
          |                                   |
          v                                   v
+-------------------+                 +--------------------+
| EMQX Server Cert  |                 | User Client Cert   |
| (Server Auth)     |                 | (Client Auth)      |
+-------------------+                 +--------------------+
```

创建一个 DesignLibroCA 作为根证书

![](images/EJBCA证书平台操作手册-image-41.png)

![](images/EJBCA证书平台操作手册-image-42.png)

创建一个 PetLibroCA 作为中间证书，编辑并且选择 DesignLibro 作为根证书，自己作为子证书

![](images/EJBCA证书平台操作手册-image-43.png)

EMQX 配置证书要配置完整的证书链

![](images/EJBCA证书平台操作手册-image-44.png)

**MQTT-CLIENT**

![](images/EJBCA证书平台操作手册-image-45.png)

![](images/EJBCA证书平台操作手册-image-46.png)

**MQTT-SERVER**&#x20;

![](images/EJBCA证书平台操作手册-image-47.png)

![](images/EJBCA证书平台操作手册-image-48.png)

**终端实体配置**

![](images/EJBCA证书平台操作手册-image-49.png)

![](images/EJBCA证书平台操作手册-image-50.png)

![](images/EJBCA证书平台操作手册-image-51.png)

## 4.3 证书吊销 CRL 配置

![](images/EJBCA证书平台操作手册-image-52.png)

![](images/EJBCA证书平台操作手册-image-53.png)

![](images/EJBCA证书平台操作手册-image-54.png)

创建Client、Server证书配置

![](images/EJBCA证书平台操作手册-image-55.png)

MQTT-CLIENT &#x20;

## 4.4 证书吊销 OCSP 配置



# 五、 EJBCA RA 配置

## 5.1 MQTT 服务器鉴权配置





# 六、证书使用（EMQX）

## 6.1 EMQX 配置 TLS 证书

> EMQX : 4.4.19

![](images/EJBCA证书平台操作手册-image-56.png)

## 6.2 EMQX OSCP配置

> EMQX : 4.4.19

![](images/EJBCA证书平台操作手册-image-57.png)



![](images/EJBCA证书平台操作手册-image-58.png)

![](images/EJBCA证书平台操作手册-image-59.png)

```shell
openssl ocsp -issuer /Users/ga666666/Desktop/ejbca-cert/dev/ManagementCA.pem -cert /Users/ga666666/Desktop/ejbca-cert/dev/user/cert.pem -url https://192.168.10.106:9443/ejbca/publicweb/status/ocsp -CAfile /Users/ga666666/Desktop/ejbca-cert/dev/ManagementCA.pem
```

## 6.3 EMQX CRL 配置

> EMQX : 5.8.4

![](images/EJBCA证书平台操作手册-image-60.png)



![](images/EJBCA证书平台操作手册-image-61.png)

![](images/EJBCA证书平台操作手册-image-62.png)

![](images/EJBCA证书平台操作手册-image-63.png)

![](images/EJBCA证书平台操作手册-image-64.png)

此时用已注销证书无法进行登录



