# Spring Boot 3 现代化升级之路

## 概述

本文档记录了从 Spring Boot 2.x + JDK 11/17 升级到 Spring Boot 3.x + JDK 21 的完整过程，涵盖自动化工具使用、依赖升级、代码迁移、常见问题解决等各个方面。

## 1. 升级前准备

### 1.1 环境要求

- **JDK 21**：确保已安装并配置
- **Maven 3.6+**：用于构建和依赖管理
- **IDE 支持**：确保 IDE 支持 JDK 21（如 IntelliJ IDEA 2023.1+）

### 1.2 中间件版本要求

升级前需要确认以下中间件版本兼容性：

- **Nacos**：需要升级至 2.5 及以上版本（推荐使用最新版本 3.1.1+）
- **Elasticsearch**：如使用 ES 7.x，需要特殊配置（详见下文）
- **Redis**：确保版本兼容 Spring Boot 3.x

### 1.3 备份与分支管理

- 创建专门的升级分支
- 备份当前代码和配置文件
- 记录当前依赖版本清单

## 2. 使用 OpenRewrite 自动化升级

OpenRewrite 是一个强大的代码重构和迁移工具，可以自动化处理大部分升级工作。

### 2.1 添加 Maven 插件

在目标工程的 `pom.xml` 的 `<plugins>` 中添加：

```xml
<plugin>
    <groupId>org.openrewrite.maven</groupId>
    <artifactId>rewrite-maven-plugin</artifactId>
    <version>6.25.0</version>
    <configuration>
        <activeRecipes>
            <!-- 升级到 Java 21 -->
            <recipe>org.openrewrite.java.migrate.UpgradeToJava21</recipe>
            <!-- 升级到 Spring Boot 3.4 -->
            <recipe>org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_4</recipe>
            <!-- 升级到 Spring Cloud 2024 -->
            <recipe>org.openrewrite.java.spring.cloud2024.UpgradeSpringCloud_2024</recipe>
        </activeRecipes>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.openrewrite.recipe</groupId>
            <artifactId>rewrite-spring</artifactId>
            <version>6.20.0</version>
        </dependency>
    </dependencies>
</plugin>
```

### 2.2 执行预览（Dry Run）

使用 **Java 11 环境**运行预览命令，查看需要修改的内容：

```bash
mvn rewrite:dryRun
```

运行结果会生成 `.patch` 文件，展示需要修改的代码差异。仔细检查这些变更，确保符合预期。

### 2.3 执行自动升级

确认预览无误后，执行实际升级：

```bash
mvn rewrite:run
```

该命令会自动修改代码中的相关部分。

### 2.4 清理插件

升级完成后，记得删除添加的 OpenRewrite 插件，避免影响后续构建。

### 2.5 常用 Recipe 说明

- `org.openrewrite.java.migrate.UpgradeToJava21`：升级到 Java 21
- `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_3`：升级到 Spring Boot 3.3
- `org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_4`：升级到 Spring Boot 3.4
- `org.openrewrite.java.spring.cloud2024.UpgradeSpringCloud_2024`：升级到 Spring Cloud 2024

## 3. 核心框架版本升级

### 3.1 Spring 生态系统版本

| 组件 | 旧版本 | 新版本 | 说明 |
|------|--------|--------|------|
| Spring Boot | 2.7.x | 3.4.12 | 核心框架 |
| Spring Cloud | 2021.x | 2024.0.0 | 微服务框架 |
| Spring Cloud Alibaba | 2021.x | 2023.0.1.2 | 阿里云组件 |

### 3.2 Jakarta EE 迁移

Spring Boot 3.x **强制使用 Jakarta EE 命名空间**，所有 `javax.*` 包需要迁移到 `jakarta.*`：

| 旧包名 | 新包名 |
|--------|--------|
| `javax.servlet.*` | `jakarta.servlet.*` |
| `javax.validation.*` | `jakarta.validation.*` |
| `javax.persistence.*` | `jakarta.persistence.*` |
| `javax.annotation.*` | `jakarta.annotation.*` |

**代码示例：**

```java
// 旧代码
import javax.servlet.http.HttpServletRequest;
import javax.validation.constraints.NotNull;

// 新代码
import jakarta.servlet.http.HttpServletRequest;
import jakarta.validation.constraints.NotNull;
```

OpenRewrite 会自动处理大部分命名空间迁移，但建议手动检查确保完整性。

## 4. 关键依赖变更

### 4.1 Knife4j API 文档（Swagger）

**变更原因**：Spring Boot 3.x 使用 Jakarta Servlet API，需要使用 Knife4j 的 Jakarta 兼容版本。

**旧依赖：**
```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

**新依赖：**
```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.5.0</version>
</dependency>
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

### 4.2 Sa-Token 认证框架

**变更原因**：Spring Boot 3.x 需要使用专门的 spring-boot3-starter。

**新依赖：**
```xml
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot3-starter</artifactId>
    <version>1.44.0</version>
</dependency>
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-fastjson2</artifactId>
    <version>1.44.0</version>
</dependency>
```

### 4.3 MyBatis-Plus / MyBatis-Spring

**变更原因**：Spring Boot 3.4.x 需要 `mybatis-spring` 3.0.4+ 版本。

**新依赖：**
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.4</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.16</version>
</dependency>
```

### 4.4 PageHelper 分页插件

**配置示例：**
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>2.1.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 4.5 Spring Data Elasticsearch（重大变更）

**变更原因**：Spring Boot 3.x 中 Elasticsearch 客户端 API 发生重大变化，`RestHighLevelClient` 被废弃，改用新的 `ElasticsearchClient`。

#### 4.5.1 依赖配置

```xml
<!-- Spring Data Elasticsearch -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
<!-- Elasticsearch Low-Level REST Client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
</dependency>
<!-- Elasticsearch Java Client (新版 API) -->
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
</dependency>
<!-- Jackson for ES JSON mapping -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

#### 4.5.2 排除自动配置（兼容 ES 7.x）

Spring Boot 3.x 默认的 ES 自动配置针对 ES 8.x，连接 ES 7.x 集群需要自定义配置。

**启动类配置：**
```java
import org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchClientAutoConfiguration;
import org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration;

@SpringBootApplication(exclude = {
    // 排除 ES 自动配置，使用自定义 ElasticsearchConfig 兼容 ES 7.x
    ElasticsearchClientAutoConfiguration.class,
    ElasticsearchRestClientAutoConfiguration.class
})
public class Application {
    // ...
}
```

#### 4.5.3 配置属性类

```java
package com.example.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import java.util.ArrayList;
import java.util.List;

@Data
@Component
@ConfigurationProperties(prefix = "spring.elasticsearch.rest")
public class ElasticsearchProperties {

    /** ES 集群地址列表 */
    private List<String> uris = new ArrayList<>();

    /** 用户名 */
    private String username;

    /** 密码 */
    private String password;

    /** 连接超时（毫秒） */
    private int connectionTimeout = 5000;

    /** Socket 超时（毫秒） */
    private int socketTimeout = 60000;
}
```

#### 4.5.4 自定义配置类（兼容 ES 7.x/8.x）

```java
package com.example.config;

import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.ElasticsearchTransport;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpHost;
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Lazy;
import org.springframework.context.annotation.Primary;
import org.springframework.data.elasticsearch.client.elc.ElasticsearchTemplate;
import org.springframework.data.elasticsearch.core.ElasticsearchOperations;
import org.springframework.data.elasticsearch.core.convert.ElasticsearchConverter;
import org.springframework.data.elasticsearch.core.convert.MappingElasticsearchConverter;
import org.springframework.data.elasticsearch.core.mapping.SimpleElasticsearchMappingContext;

import java.util.List;

@Slf4j
@Configuration
@Lazy  // 延迟初始化，确保配置已加载
@RequiredArgsConstructor
public class ElasticsearchConfig {
    
    private final ElasticsearchProperties elasticsearchProperties;

    @Bean
    @Primary
    public RestClient elasticsearchRestClient() {
        List<String> uris = elasticsearchProperties.getUris();
        HttpHost[] httpHosts = parseHosts(uris);

        RestClientBuilder builder = RestClient.builder(httpHosts)
            .setRequestConfigCallback(requestConfigBuilder ->
                requestConfigBuilder
                    .setConnectTimeout(elasticsearchProperties.getConnectionTimeout())
                    .setSocketTimeout(elasticsearchProperties.getSocketTimeout())
            );

        // 认证配置
        String username = elasticsearchProperties.getUsername();
        if (username != null && !username.isEmpty()) {
            BasicCredentialsProvider credentialsProvider = new BasicCredentialsProvider();
            credentialsProvider.setCredentials(AuthScope.ANY,
                new UsernamePasswordCredentials(username, elasticsearchProperties.getPassword()));
            builder.setHttpClientConfigCallback(httpClientBuilder ->
                httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider));
        }

        return builder.build();
    }

    @Bean
    @Primary
    public ElasticsearchTransport elasticsearchTransport(RestClient restClient) {
        return new RestClientTransport(restClient, new JacksonJsonpMapper());
    }

    @Bean
    @Primary
    public ElasticsearchClient elasticsearchClient(ElasticsearchTransport transport) {
        return new ElasticsearchClient(transport);
    }

    @Bean
    @Primary
    public ElasticsearchConverter elasticsearchConverter() {
        return new MappingElasticsearchConverter(new SimpleElasticsearchMappingContext());
    }

    @Bean
    @Primary
    public ElasticsearchOperations elasticsearchOperations(
            ElasticsearchClient elasticsearchClient,
            ElasticsearchConverter elasticsearchConverter) {
        return new ElasticsearchTemplate(elasticsearchClient, elasticsearchConverter);
    }

    private HttpHost[] parseHosts(List<String> urisList) {
        return urisList.stream()
            .map(uri -> {
                try {
                    java.net.URI parsed = java.net.URI.create(uri);
                    return new HttpHost(parsed.getHost(), parsed.getPort(), parsed.getScheme());
                } catch (Exception e) {
                    throw new IllegalArgumentException("Invalid Elasticsearch URI: " + uri, e);
                }
            })
            .toArray(HttpHost[]::new);
    }
}
```

#### 4.5.5 YAML 配置格式

```yaml
spring:
  elasticsearch:
    rest:
      uris:
        - http://localhost:9200
      username: elastic
      password: your_password
```

#### 4.5.6 代码迁移示例

**1. 注入方式变更**

```java
// 旧代码 - 使用 ElasticsearchRestTemplate
@Autowired
private ElasticsearchRestTemplate elasticsearchRestTemplate;

// 新代码 - 使用 ElasticsearchOperations
@Autowired
private ElasticsearchOperations elasticsearchOperations;
```

**2. 查询构建器变更**

```java
// 旧代码 - 使用旧版 QueryBuilders
import org.elasticsearch.index.query.QueryBuilders;
import org.springframework.data.elasticsearch.core.query.NativeSearchQueryBuilder;

NativeSearchQuery query = new NativeSearchQueryBuilder()
    .withQuery(QueryBuilders.matchAllQuery())
    .withCollapse(new CollapseBuilder("fieldName"))
    .build();

// 新代码 - 使用新版 co.elastic.clients API
import co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders;
import org.springframework.data.elasticsearch.client.elc.NativeQuery;
import org.springframework.data.elasticsearch.client.elc.NativeQueryBuilder;
import co.elastic.clients.elasticsearch.core.search.FieldCollapse;

FieldCollapse fieldCollapse = FieldCollapse.of(c -> c.field("fieldName"));

NativeQuery query = new NativeQueryBuilder()
    .withQuery(q -> q.matchAll(m -> m))
    .withFieldCollapse(fieldCollapse)
    .build();
```

**3. 范围查询变更**

```java
// 旧代码
QueryBuilders.rangeQuery("timestamp")
    .gte(startTime)
    .lte(endTime)

// 新代码 - Lambda 风格
import static co.elastic.clients.elasticsearch._types.query_dsl.QueryBuilders.*;
import co.elastic.clients.json.JsonData;

range(r -> r
    .field("timestamp")
    .gte(JsonData.of(startTime))
    .lte(JsonData.of(endTime))
)
```

**4. 布尔查询变更**

```java
// 旧代码
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery()
    .must(QueryBuilders.termQuery("status", "active"))
    .filter(QueryBuilders.rangeQuery("date").gte(startDate));

// 新代码
NativeQuery query = new NativeQueryBuilder()
    .withQuery(q -> q.bool(b -> b
        .must(m -> m.term(t -> t.field("status").value("active")))
        .filter(f -> f.range(r -> r.field("date").gte(JsonData.of(startDate))))
    ))
    .build();
```

**5. 执行查询变更**

```java
// 旧代码
SearchHits<MyDoc> hits = elasticsearchRestTemplate.search(query, MyDoc.class);

// 新代码
SearchHits<MyDoc> hits = elasticsearchOperations.search(
    query,
    MyDoc.class,
    IndexCoordinates.of("index_name")
);
```

**6. 排序变更**

```java
// 旧代码
.withSort(SortBuilders.fieldSort("timestamp").order(SortOrder.DESC))

// 新代码
import co.elastic.clients.elasticsearch._types.SortOptions;
import co.elastic.clients.elasticsearch._types.SortOrder;

.withSort(SortOptions.of(s -> s.field(f -> f.field("timestamp").order(SortOrder.Desc))))
```

## 5. 代码修改

### 5.1 启动类修改

**修改原因**：某些第三方库（如 SOFA）的日志系统可能与 Spring Boot 3.x 的 Logback 1.5.x 版本冲突。

```java
public class Application {
    public static void main(String[] args) {
        // 禁用第三方日志系统，避免与 Spring Boot 3.x 的 Logback 冲突
        System.setProperty("sofa.middleware.log.disable", "true");
        
        SpringApplication.run(Application.class, args);
    }
}
```

### 5.2 PageUtils 工具类修改

**修改原因**：JDK 21 下 `Convert.toList()` 方法行为变化，需要使用 `BeanUtil.copyProperties` 替代。

**旧代码：**
```java
public static <T, E> PageResultDTO<T> convert(Page<E> page, Class<T> clazz) {
    return new PageResultDTO<T>(
        page.getPageSize(), 
        page.getPageNum(), 
        page.getPages(), 
        page.getTotal(), 
        Convert.toList(clazz, page.getResult())  // JDK 21 下可能失败
    );
}
```

**新代码：**
```java
public static <T, E> PageResultDTO<T> convert(Page<E> page, Class<T> clazz) {
    List<T> list = page.getResult().stream()
        .map(e -> {
            try {
                T t = clazz.getDeclaredConstructor().newInstance();
                BeanUtil.copyProperties(e, t);
                return t;
            } catch (Exception ex) {
                throw new RuntimeException("Failed to convert object", ex);
            }
        })
        .collect(Collectors.toList());
    return new PageResultDTO<T>(
        page.getPageSize(), 
        page.getPageNum(), 
        page.getPages(), 
        page.getTotal(), 
        list
    );
}
```

### 5.3 javax.* → jakarta.* 命名空间迁移

所有使用 `javax.servlet.*` 和 `javax.validation.*` 的代码需要迁移到 Jakarta 命名空间。OpenRewrite 会自动处理大部分迁移，但建议全局搜索确保完整性：

```bash
# 搜索所有 javax 包的使用
grep -r "import javax\." src/
```

## 6. 配置修改

### 6.1 JVM 参数配置（JDK 21 模块系统兼容）

在 `pom.xml` 的 `spring-boot-maven-plugin` 中配置：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!-- JDK 21 模块系统兼容性 -->
        <jvmArguments>
            --add-opens=java.base/java.lang=ALL-UNNAMED
            --add-opens=java.base/java.util=ALL-UNNAMED
            --add-opens=java.base/java.lang.reflect=ALL-UNNAMED
            --add-opens=java.base/java.text=ALL-UNNAMED
            --add-opens=java.desktop/java.awt.font=ALL-UNNAMED
        </jvmArguments>
    </configuration>
</plugin>
```

### 6.2 SpringDoc 配置（排除 AWS SDK 类扫描）

**问题**：SpringDoc 可能扫描到 AWS SDK 的类导致冲突。

**解决方案**：在 `application.yml` 或 `bootstrap.yml` 中配置：

```yaml
springdoc:
  # 排除 AWS SDK 类，避免扫描冲突
  packages-to-exclude: com.amazonaws.**
  model-converters:
    deprecating-converter:
      enabled: false
```

## 7. 常见问题与解决方案

### 7.1 BeanDefinitionStoreException: Invalid bean definition

**错误信息：**
```
Invalid bean definition with name 'xxxMapper' defined in file [.../XxxMapper.class]: 
Invalid value type for attribute 'factoryBeanObjectType': java.lang.String
```

**原因**：`mybatis-spring` 版本过低，与 Spring Boot 3.4.x 不兼容。

**解决方案**：升级 `mybatis-spring` 到 3.0.4：

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.4</version>
</dependency>
```

### 7.2 NoSuchMethodError: ContextInitializer.configureByResource

**错误信息：**
```
java.lang.NoSuchMethodError: 'void ch.qos.logback.classic.util.ContextInitializer.configureByResource(java.net.URL)'
```

**原因**：第三方库（如 SOFA）的日志系统与 Spring Boot 3.x 的 Logback 1.5.x 版本冲突。

**解决方案**：在启动类中禁用第三方日志：

```java
System.setProperty("sofa.middleware.log.disable", "true");
```

### 7.3 SpringDoc 扫描 AWS SDK 类冲突

**错误信息：**
```
IllegalArgumentException: Conflicting setter definitions for property "objectContent"
```

**原因**：SpringDoc 扫描到 AWS SDK 的类导致冲突。

**解决方案**：配置排除 AWS 包：

```yaml
springdoc:
  packages-to-exclude: com.amazonaws.**
```

### 7.4 Convert.toList 转换失败

**错误信息：**
```
RuntimeException: Failed to convert object from ... to ...
```

**原因**：JDK 21 下 Hutool `Convert.toList()` 方法行为变化。

**解决方案**：使用 `BeanUtil.copyProperties` + Stream 替代 `Convert.toList()`（详见 5.2 节）。

### 7.5 编译错误: 不支持发行版本 21

**错误信息：**
```
错误: 不支持发行版本 21
```

**原因**：Maven 环境使用的 JDK 版本不是 21。

**解决方案：**
1. 确保系统安装 JDK 21
2. 配置 `JAVA_HOME` 指向 JDK 21
3. IDE 中配置项目 SDK 为 JDK 21
4. 检查 Maven 配置：`mvn -version` 应显示 Java version 21

### 7.6 Elasticsearch 连接失败

**问题**：升级后无法连接到 Elasticsearch。

**解决方案：**
1. 确认 ES 版本（7.x 需要特殊配置，详见 4.5 节）
2. 检查是否排除了自动配置
3. 验证配置属性类是否正确加载
4. 检查网络连接和认证信息

## 8. 升级检查清单

### 8.1 环境配置

- [ ] JDK 21 已安装并配置
- [ ] Maven 配置使用 JDK 21 编译
- [ ] IDE 项目 SDK 设置为 JDK 21
- [ ] JVM `--add-opens` 参数已配置
- [ ] 中间件版本已确认兼容（Nacos 2.5+、Redis、ES 等）

### 8.2 POM 配置

- [ ] Spring Boot 版本已更新到 3.4.x
- [ ] Spring Cloud 版本已更新到 2024.0.0
- [ ] `mybatis-spring` 升级到 3.0.4
- [ ] 所有依赖版本已检查兼容性
- [ ] OpenRewrite 插件已移除（如已使用）

### 8.3 Jakarta EE 迁移

- [ ] `javax.*` 包已迁移到 `jakarta.*`
- [ ] Knife4j 使用 `openapi3-jakarta` 版本
- [ ] Sa-Token 使用 `spring-boot3-starter`
- [ ] 全局搜索确认无遗漏的 `javax` 包

### 8.4 Elasticsearch 迁移（如使用）

- [ ] 启动类排除 ES 自动配置（如使用 ES 7.x）
- [ ] 添加 `ElasticsearchProperties` 配置属性类
- [ ] 添加 `ElasticsearchConfig` 自定义配置类（`@Lazy`）
- [ ] YAML 配置使用 `spring.elasticsearch.rest.*` 格式
- [ ] Elasticsearch 代码迁移到新 API（`ElasticsearchOperations`）
- [ ] 查询代码迁移到 `co.elastic.clients` API

### 8.5 代码修改

- [ ] `PageUtils` 中 `Convert.toList` 已替换（如使用）
- [ ] 启动类添加第三方日志禁用配置（如需要）
- [ ] 所有工具类已检查 JDK 21 兼容性

### 8.6 配置修改

- [ ] JVM 参数已配置
- [ ] SpringDoc 配置已更新（如使用）
- [ ] 其他配置文件已检查

### 8.7 验证

- [ ] 应用成功启动无错误
- [ ] Elasticsearch 连接正常（如使用）
- [ ] API 文档可正常访问（如使用 Swagger/Knife4j）
- [ ] 数据库连接正常
- [ ] Redis 连接正常
- [ ] 核心功能测试通过
- [ ] 性能测试通过

## 9. 版本兼容性参考

### 9.1 Spring Boot 3.x 兼容性矩阵

| Spring Boot | JDK | Spring Cloud | Spring Cloud Alibaba |
|-------------|-----|--------------|---------------------|
| 3.0.x | 17+ | 2022.0.x | 2022.0.0.0 |
| 3.1.x | 17+ | 2022.0.x | 2022.0.0.0 |
| 3.2.x | 17+ | 2023.0.x | 2023.0.1.x |
| 3.3.x | 17+ | 2023.0.x | 2023.0.1.x |
| 3.4.x | 21+ | 2024.0.x | 2023.0.1.x |

### 9.2 推荐版本组合

- **Spring Boot 3.4.12** + **JDK 21** + **Spring Cloud 2024.0.0** + **Spring Cloud Alibaba 2023.0.1.2**

## 10. 总结

Spring Boot 3.x + JDK 21 升级是一个系统性的工程，涉及：

1. **自动化工具**：使用 OpenRewrite 可以大幅减少手动工作量
2. **依赖升级**：核心框架和第三方库都需要升级到兼容版本
3. **命名空间迁移**：Jakarta EE 迁移是必须的
4. **API 变更**：Elasticsearch 等组件的 API 发生了重大变化
5. **兼容性处理**：JDK 21 的模块系统需要特殊配置

建议按照本文档的顺序逐步进行，每个步骤完成后进行验证，确保问题及时发现和解决。升级完成后，建议进行全面的功能测试和性能测试，确保系统稳定运行。

## 参考资料

- [Spring Boot 3.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- [OpenRewrite Documentation](https://docs.openrewrite.org/)
- [Jakarta EE 9 Migration Guide](https://jakarta.ee/specifications/platform/9/)
- [Elasticsearch Java API Client](https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html)

