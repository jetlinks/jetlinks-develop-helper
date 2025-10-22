# JetLinks 业务模块创建指南

本指南提供了在 JetLinks 项目中创建新业务模块的完整流程和规范。

## 一、模块结构概述

JetLinks 项目采用模块化架构，所有业务模块位于 `modules/` 目录下。根据模块是否需要提供跨服务调用，模块结构分为两种形式：

### 1.1 单模块结构（简单场景）

适用于不需要跨服务调用的简单业务模块：

```
jetlinks-ultimate/
├── modules/
│   ├── {模块名}-manager/          # 业务模块目录
│   │   ├── pom.xml                # Maven配置文件
│   │   └── src/
│   │       ├── main/
│   │       │   ├── java/
│   │       │   │   └── org/jetlinks/pro/{模块名}/
│   │       │   │       ├── entity/        # 实体类
│   │       │   │       ├── enums/         # 枚举类
│   │       │   │       ├── service/       # 服务类
│   │       │   │       ├── web/           # 控制器
│   │       │   │       ├── assets/        # 资产类型定义(可选)
│   │       │   │       ├── events/        # 事件定义(可选)
│   │       │   │       └── configuration/ # 配置类(可选)
│   │       │   └── resources/
│   │       │       ├── META-INF/
│   │       │       │   └── spring/
│   │       │       │       └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
│   │       │       └── i18n/
│   │       │           └── {模块名}-manager/
│   │       │               ├── messages_zh.properties
│   │       │               └── messages_en.properties
│   │       └── test/
│   │           └── java/
│   │               └── org/jetlinks/pro/{模块名}/
│   │                   ├── service/       # 服务测试
│   │                   └── web/           # 控制器测试
```

### 1.2 多模块结构（跨服务调用场景，推荐）

**适用场景**: 当模块需要提供跨服务调用时（默认推荐使用此结构），应创建两个子模块。

**命名规范**：
- 父模块：`{项目简写}-{模块名}`（如 `pms-report`）
- API 模块：`{模块名}-api`（如 `report-api`）
- Manager 模块：`{模块名}-manager`（如 `report-manager`，实现模块）

```
jetlinks-ultimate/
├── modules/
│   ├── {项目简写}-{模块名}/              # 父模块目录（如 pms-report）
│   │   ├── pom.xml                       # 父模块 POM（聚合子模块）
│   │   │
│   │   ├── {模块名}-api/                 # API 模块（对外接口定义，如 report-api）
│   │   │   ├── pom.xml
│   │   │   └── src/
│   │   │       ├── main/
│   │   │       │   └── java/
│   │   │       │       └── org/jetlinks/pro/{模块名}/api/
│   │   │       │           ├── entity/        # 实体类（DTO、VO）
│   │   │       │           ├── enums/         # 枚举类
│   │   │       │           ├── command/       # 命令接口
│   │   │       │           ├── query/         # 查询接口
│   │   │       │           ├── event/         # 事件定义
│   │   │       │           └── constants/     # 常量定义
│   │   │       └── test/
│   │   │           └── java/
│   │   │
│   │   └── {模块名}-manager/            # Manager 实现模块（如 report-manager）
│   │       ├── pom.xml
│   │       └── src/
│   │           ├── main/
│   │           │   ├── java/
│   │           │   │   └── org/jetlinks/pro/{模块名}/
│   │           │   │       ├── entity/        # 实体实现类
│   │           │   │       ├── service/       # 服务实现类
│   │           │   │       ├── web/           # 控制器
│   │           │   │       ├── command/       # 命令实现
│   │           │   │       ├── assets/        # 资产类型定义(可选)
│   │           │   │       └── configuration/ # 配置类
│   │           │   └── resources/
│   │           │       ├── META-INF/
│   │           │       │   └── spring/
│   │           │       │       └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
│   │           │       └── i18n/
│   │           │           └── {模块名}-manager/
│   │           │               ├── messages_zh.properties
│   │           │               └── messages_en.properties
│   │           └── test/
│   │               └── java/
│   │                   └── org/jetlinks/pro/{模块名}/
│   │                       ├── service/       # 服务测试
│   │                       └── web/           # 控制器测试
```

#### 多模块结构说明：

1. **{模块名}-api 模块**：
   - 仅包含接口定义、实体类（DTO/VO）、枚举、常量等
   - 不包含具体实现逻辑
   - 可被其他服务依赖，用于跨服务调用
   - 依赖项尽可能少，只包含必要的基础依赖

2. **{模块名}-manager 模块**（实现模块）：
   - 包含所有业务逻辑实现
   - 依赖 `{模块名}-api` 模块
   - 包含控制器、服务实现、数据库操作等
   - 包含配置类、资源文件、测试代码等

3. **父模块 `{项目简写}-{模块名}`**：
   - 聚合 api 和 manager 两个子模块
   - 统一管理版本号和公共配置
   - 项目简写如：pms（平台管理系统）、dms（设备管理系统）等

---

## 二、创建模块步骤

### 2.1 明确模块需求

在创建模块前，需要明确以下信息：

#### 基础信息
- **模块名称**: 如 `report`, `notify`, `workflow`（使用短横线命名，如 `rule-engine`）
- **项目简写**: 如 `pms`（平台管理系统）、`dms`（设备管理系统）等
- **模块描述**: 简要说明模块用途
- **包名**: 通常为 `org.jetlinks.pro.{模块名}`
- **artifactId**: 
  - 单模块：`{模块名}-manager`（如 `template-manager`）
  - 多模块：
    - 父模块：`{项目简写}-{模块名}`（如 `pms-report`）
    - API 子模块：`{模块名}-api`（如 `report-api`）
    - Manager 子模块：`{模块名}-manager`（如 `report-manager`）
- **模块结构选择**: 
  - **单模块**: 简单业务，不需要跨服务调用
  - **多模块（推荐）**: 需要提供跨服务调用接口，或模块较复杂需要拆分

#### 技术选型
- **编程模式**: 
  - 响应式(WebFlux): 适合高并发、IO密集型场景
  - 阻塞式(Spring MVC): 适合传统业务、低并发场景
- **数据存储**: 
  - 关系型数据库(PostgreSQL/MySQL)
  - 时序数据库(InfluxDB/TDengine)
  - ElasticSearch
  - MongoDB

#### 业务特性
- **核心实体**: 列出主要的实体类及其字段
- **权限控制**: 是否需要资产权限控制
- **资产类型**: 如果需要，定义资产类型
- **依赖组件**: 列出需要依赖的其他组件
- **跨服务调用**: 是否需要被其他服务调用（推荐使用多模块结构）

---

### 2.2 选择模块结构

根据业务需求选择合适的模块结构：

#### 使用单模块结构的场景：
- ✅ 简单的 CRUD 业务
- ✅ 不需要被其他服务依赖
- ✅ 模块代码量较小（< 50 个类）
- ✅ 快速原型开发

#### 使用多模块结构的场景（推荐）：
- ✅ 需要提供跨服务调用接口
- ✅ 需要在微服务架构中被其他服务依赖
- ✅ 模块较复杂，需要清晰的接口定义
- ✅ 需要版本化管理 API
- ✅ 需要限制其他服务对内部实现的依赖

> **建议**: 对于新建模块，默认使用多模块结构，即使暂时不需要跨服务调用，这样可以为未来扩展留下空间。

---

### 2.3 创建 pom.xml

根据选择的模块结构，创建对应的 POM 文件。

#### 方式一：单模块 POM（简单场景）

适用于不需要跨服务调用的简单业务模块：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 父项目配置 -->
    <parent>
        <groupId>org.jetlinks.pro</groupId>
        <artifactId>jetlinks-parent</artifactId>
        <version>2.11.0-SNAPSHOT</version>
        <relativePath>../../jetlinks-parent</relativePath>
    </parent>

    <!-- 当前模块的 artifactId -->
    <artifactId>{模块名}-manager</artifactId>

    <dependencies>
        <!-- ========== 核心依赖(必需) ========== -->
        
        <!-- 通用组件: 包含基础工具类、实体类等 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>common-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- HSWeb Web 框架: 提供 CRUD、权限控制等基础功能 -->
        <dependency>
            <groupId>org.hswebframework.web</groupId>
            <artifactId>hsweb-starter</artifactId>
            <version>${hsweb.framework.version}</version>
        </dependency>

        <!-- HSWeb 权限 API -->
        <dependency>
            <groupId>org.hswebframework.web</groupId>
            <artifactId>hsweb-authorization-api</artifactId>
            <version>${hsweb.framework.version}</version>
        </dependency>

        <!-- ========== 数据库相关 ========== -->

        <!-- EasyORM: 响应式/阻塞式 ORM 框架 -->
        <dependency>
            <groupId>org.hswebframework</groupId>
            <artifactId>hsweb-easy-orm-rdb</artifactId>
        </dependency>

        <!-- R2DBC H2: 用于内存数据库(开发/测试) -->
        <dependency>
            <groupId>io.r2dbc</groupId>
            <artifactId>r2dbc-h2</artifactId>
        </dependency>

        <!-- ========== 资产权限相关(可选) ========== -->
        
        <!-- 资产组件: 如需要资产权限控制，添加此依赖 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>assets-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- ========== 其他常用组件(按需添加) ========== -->

        <!-- JetLinks SDK API -->
        <dependency>
            <groupId>org.jetlinks.sdk</groupId>
            <artifactId>jetlinks-sdk-api</artifactId>
        </dependency>

        <!-- JetLinks 核心 -->
        <dependency>
            <groupId>org.jetlinks</groupId>
            <artifactId>jetlinks-core</artifactId>
            <version>${jetlinks.version}</version>
        </dependency>

        <!-- JetLinks 支持库 -->
        <dependency>
            <groupId>org.jetlinks</groupId>
            <artifactId>jetlinks-supports</artifactId>
            <version>${jetlinks.version}</version>
        </dependency>

        <!-- 时序数据组件 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>timeseries-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- ElasticSearch 核心(可选) -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>elasticsearch-core</artifactId>
            <version>${project.version}</version>
            <optional>true</optional>
        </dependency>

        <!-- 关系组件: 实体关系管理 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>relation-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- 网关组件: API 网关相关 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>gateway-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- 通知组件核心 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>notify-core</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- 规则引擎组件 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>rule-engine-component</artifactId>
            <version>${project.version}</version>
            <optional>true</optional>
        </dependency>

        <!-- 脚本组件 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>script-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- API组件: OpenAPI 相关 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>api-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- 应用组件: 应用管理 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>application-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- 数据字典 -->
        <dependency>
            <groupId>org.hswebframework.web</groupId>
            <artifactId>hsweb-system-dictionary</artifactId>
        </dependency>

        <!-- ========== 测试依赖 ========== -->

        <!-- 测试组件 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>test-component</artifactId>
            <version>${project.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Spring Data Redis (测试用) -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <!-- Maven 仓库配置 -->
    <repositories>
        <repository>
            <id>jetlinks</id>
            <name>JetLinks Maven Repository</name>
            <url>https://nexus.jetlinks.cn/content/groups/jetlinks/</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </repository>
    </repositories>
</project>
```

#### 方式二：多模块 POM（跨服务调用场景，推荐）

适用于需要提供跨服务调用的模块，将模块拆分为 API 和 Manager 两个子模块。

##### 1. 父模块 POM (modules/{项目简写}-{模块名}/pom.xml)

示例：`modules/pms-report/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 父项目配置 -->
    <parent>
        <groupId>org.jetlinks.pro</groupId>
        <artifactId>jetlinks-parent</artifactId>
        <version>2.11.0-SNAPSHOT</version>
        <relativePath>../../jetlinks-parent</relativePath>
    </parent>

    <!-- 父模块 artifactId: {项目简写}-{模块名} -->
    <artifactId>pms-report</artifactId>
    <packaging>pom</packaging>
    <name>JetLinks Pro Report Module</name>
    <description>报表管理模块</description>

    <!-- 聚合子模块 -->
    <modules>
        <module>report-api</module>
        <module>report-manager</module>
    </modules>

    <!-- Maven 仓库配置 -->
    <repositories>
        <repository>
            <id>jetlinks</id>
            <name>JetLinks Maven Repository</name>
            <url>https://nexus.jetlinks.cn/content/groups/jetlinks/</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </repository>
    </repositories>
</project>
```

##### 2. API 模块 POM (modules/{项目简写}-{模块名}/{模块名}-api/pom.xml)

示例：`modules/pms-report/report-api/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 父模块配置 -->
    <parent>
        <groupId>org.jetlinks.pro</groupId>
        <artifactId>pms-report</artifactId>
        <version>2.11.0-SNAPSHOT</version>
    </parent>

    <!-- API 模块 artifactId: {模块名}-api -->
    <artifactId>report-api</artifactId>
    <name>JetLinks Pro Report API</name>
    <description>报表管理模块 - API 接口定义</description>

    <dependencies>
        <!-- ========== 最小化依赖（仅包含必要的基础依赖）========== -->
        
        <!-- Lombok: 减少样板代码 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>

        <!-- Reactor Core: 响应式编程（如果使用响应式） -->
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
            <scope>provided</scope>
        </dependency>

        <!-- Swagger 注解: API 文档 -->
        <dependency>
            <groupId>io.swagger.core.v3</groupId>
            <artifactId>swagger-annotations-jakarta</artifactId>
            <scope>provided</scope>
        </dependency>

        <!-- Validation API: 参数验证 -->
        <dependency>
            <groupId>jakarta.validation</groupId>
            <artifactId>jakarta.validation-api</artifactId>
        </dependency>

        <!-- Jackson 注解: JSON 序列化 -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
        </dependency>

        <!-- JetLinks SDK API: 如需要设备相关定义 -->
        <dependency>
            <groupId>org.jetlinks.sdk</groupId>
            <artifactId>jetlinks-sdk-api</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

##### 3. Manager 实现模块 POM (modules/{项目简写}-{模块名}/{模块名}-manager/pom.xml)

示例：`modules/pms-report/report-manager/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 父模块配置 -->
    <parent>
        <groupId>org.jetlinks.pro</groupId>
        <artifactId>pms-report</artifactId>
        <version>2.11.0-SNAPSHOT</version>
    </parent>

    <!-- Manager 模块 artifactId: {模块名}-manager -->
    <artifactId>report-manager</artifactId>
    <name>JetLinks Pro Report Manager</name>
    <description>报表管理模块 - 服务实现</description>

    <dependencies>
        <!-- ========== API 模块依赖 ========== -->
        
        <!-- 依赖本模块的 API -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>report-api</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- ========== 核心依赖(必需) ========== -->
        
        <!-- 通用组件: 包含基础工具类、实体类等 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>common-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- HSWeb Web 框架: 提供 CRUD、权限控制等基础功能 -->
        <dependency>
            <groupId>org.hswebframework.web</groupId>
            <artifactId>hsweb-starter</artifactId>
            <version>${hsweb.framework.version}</version>
        </dependency>

        <!-- HSWeb 权限 API -->
        <dependency>
            <groupId>org.hswebframework.web</groupId>
            <artifactId>hsweb-authorization-api</artifactId>
            <version>${hsweb.framework.version}</version>
        </dependency>

        <!-- EasyORM: 响应式/阻塞式 ORM 框架 -->
        <dependency>
            <groupId>org.hswebframework</groupId>
            <artifactId>hsweb-easy-orm-rdb</artifactId>
        </dependency>

        <!-- R2DBC H2: 用于内存数据库(开发/测试) -->
        <dependency>
            <groupId>io.r2dbc</groupId>
            <artifactId>r2dbc-h2</artifactId>
        </dependency>

        <!-- ========== 其他常用组件(按需添加) ========== -->
        
        <!-- 资产组件: 如需要资产权限控制，添加此依赖 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>assets-component</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!-- 其他依赖参考单模块 POM 中的依赖说明 -->

        <!-- ========== 测试依赖 ========== -->

        <!-- 测试组件 -->
        <dependency>
            <groupId>org.jetlinks.pro</groupId>
            <artifactId>test-component</artifactId>
            <version>${project.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

#### 常见依赖说明

| 依赖组件 | artifactId | 用途 | 是否必需 |
|---------|-----------|------|---------|
| 通用组件 | common-component | 基础工具、实体类、服务等 | ✅ 必需 |
| HSWeb Starter | hsweb-starter | Web框架、CRUD基础功能 | ✅ 必需 |
| HSWeb 权限 API | hsweb-authorization-api | 权限控制 | ✅ 必需 |
| EasyORM | hsweb-easy-orm-rdb | 数据库ORM | ✅ 必需 |
| R2DBC H2 | r2dbc-h2 | 内存数据库(测试) | ✅ 必需 |
| 资产组件 | assets-component | 资产权限控制 | ❌ 可选 |
| JetLinks 核心 | jetlinks-core | JetLinks核心功能 | ⚠️ 按需 |
| JetLinks 支持库 | jetlinks-supports | JetLinks支持工具 | ⚠️ 按需 |
| 时序数据组件 | timeseries-component | 时序数据存储 | ⚠️ 按需 |
| ElasticSearch | elasticsearch-core | ES搜索功能 | ⚠️ 按需 |
| 关系组件 | relation-component | 实体关系管理 | ⚠️ 按需 |
| 网关组件 | gateway-component | API网关 | ⚠️ 按需 |
| 通知组件 | notify-core | 消息通知 | ⚠️ 按需 |
| 规则引擎 | rule-engine-component | 规则引擎功能 | ⚠️ 按需 |
| 脚本组件 | script-component | 脚本执行 | ⚠️ 按需 |
| API组件 | api-component | OpenAPI | ⚠️ 按需 |
| 测试组件 | test-component | 测试工具 | ✅ 必需(test) |

---

### 2.4 创建目录结构

根据选择的模块结构创建对应的目录。

#### 方式一：单模块目录结构

```bash
# 创建单模块目录结构
mkdir -p modules/{模块名}-manager/src/main/java/org/jetlinks/pro/{模块名}/{entity,service,web,enums,configuration}
mkdir -p modules/{模块名}-manager/src/main/resources/META-INF/spring
mkdir -p modules/{模块名}-manager/src/main/resources/i18n/{模块名}-manager
mkdir -p modules/{模块名}-manager/src/test/java/org/jetlinks/pro/{模块名}/{service,web}
```

#### 方式二：多模块目录结构（推荐）

示例：创建 `pms-report` 模块

```bash
# 1. 创建父模块目录（{项目简写}-{模块名}）
mkdir -p modules/pms-report

# 2. 创建 API 模块目录结构（{模块名}-api）
mkdir -p modules/pms-report/report-api/src/main/java/org/jetlinks/pro/report/api/{entity,enums,command,query,event,constants}
mkdir -p modules/pms-report/report-api/src/test/java/org/jetlinks/pro/report/api

# 3. 创建 Manager 模块目录结构（{模块名}-manager）
mkdir -p modules/pms-report/report-manager/src/main/java/org/jetlinks/pro/report/{entity,service,web,command,configuration}
mkdir -p modules/pms-report/report-manager/src/main/resources/META-INF/spring
mkdir -p modules/pms-report/report-manager/src/main/resources/i18n/report-manager
mkdir -p modules/pms-report/report-manager/src/test/java/org/jetlinks/pro/report/{service,web}
```

**通用模板命令**：

```bash
# 替换以下变量：
# - {项目简写}: 如 pms
# - {模块名}: 如 report

# 1. 创建父模块目录
mkdir -p modules/{项目简写}-{模块名}

# 2. 创建 API 模块
mkdir -p modules/{项目简写}-{模块名}/{模块名}-api/src/main/java/org/jetlinks/pro/{模块名}/api/{entity,enums,command,query,event,constants}
mkdir -p modules/{项目简写}-{模块名}/{模块名}-api/src/test/java/org/jetlinks/pro/{模块名}/api

# 3. 创建 Manager 模块
mkdir -p modules/{项目简写}-{模块名}/{模块名}-manager/src/main/java/org/jetlinks/pro/{模块名}/{entity,service,web,command,configuration}
mkdir -p modules/{项目简写}-{模块名}/{模块名}-manager/src/main/resources/META-INF/spring
mkdir -p modules/{项目简写}-{模块名}/{模块名}-manager/src/main/resources/i18n/{模块名}-manager
mkdir -p modules/{项目简写}-{模块名}/{模块名}-manager/src/test/java/org/jetlinks/pro/{模块名}/{service,web}
```

---

### 2.5 创建配置类

#### 单模块配置

##### AutoConfiguration.imports

创建文件: `src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

```
org.jetlinks.pro.{模块名}.configuration.{模块名首字母大写}ManagerConfiguration
```

##### Configuration 类

创建文件: `src/main/java/org/jetlinks/pro/{模块名}/configuration/{模块名首字母大写}ManagerConfiguration.java`

```java
package org.jetlinks.pro.{模块名}.configuration;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.hswebframework.web.crud.annotation.EnableEasyormRepository;

/**
 * {模块名称}管理模块配置
 *
 * @author {作者}
 * @since {版本}
 */
@Configuration
@EnableEasyormRepository("org.jetlinks.pro.{模块名}.entity")  // 启用实体类扫描
@Slf4j
public class {模块名首字母大写}ManagerConfiguration {

    @Bean
    public {模块名首字母大写}Properties {模块名}Properties() {
        return new {模块名首字母大写}Properties();
    }

    // 其他Bean配置...
}
```

#### 多模块配置

在多模块结构中，只需要在 Manager 模块创建配置类。

**示例：`pms-report/report-manager` 模块**

##### AutoConfiguration.imports

创建文件: `report-manager/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

```
org.jetlinks.pro.report.configuration.ReportManagerConfiguration
```

##### Configuration 类

创建文件: `report-manager/src/main/java/org/jetlinks/pro/report/configuration/ReportManagerConfiguration.java`

```java
package org.jetlinks.pro.report.configuration;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.hswebframework.web.crud.annotation.EnableEasyormRepository;

/**
 * 报表管理模块配置
 *
 * @author {作者}
 * @since {版本}
 */
@Configuration
@EnableEasyormRepository("org.jetlinks.pro.report.entity")  // 启用实体类扫描
@Slf4j
public class ReportManagerConfiguration {

    @Bean
    public ReportProperties reportProperties() {
        return new ReportProperties();
    }

    // 其他Bean配置...
}
```

**通用模板**：

```
文件路径: {模块名}-manager/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
内容: org.jetlinks.pro.{模块名}.configuration.{模块名首字母大写}ManagerConfiguration
```

---

### 2.6 创建国际化文件

#### 中文消息文件
创建: `src/main/resources/i18n/{模块名}-manager/messages_zh.properties`

```properties
# 错误消息
error.{模块名}_not_found=数据不存在
error.{模块名}_name_duplicate=名称已存在
error.{模块名}_state_invalid=状态无效

# 成功消息
success.{模块名}_save=保存成功
success.{模块名}_delete=删除成功
success.{模块名}_update=更新成功

# 验证消息
validation.{模块名}_name_required=名称不能为空
validation.{模块名}_id_invalid=ID格式无效
```

#### 英文消息文件
创建: `src/main/resources/i18n/{模块名}-manager/messages_en.properties`

```properties
# Error messages
error.{模块名}_not_found=Data not found
error.{模块名}_name_duplicate=Name already exists
error.{模块名}_state_invalid=Invalid state

# Success messages
success.{模块名}_save=Saved successfully
success.{模块名}_delete=Deleted successfully
success.{模块名}_update=Updated successfully

# Validation messages
validation.{模块名}_name_required=Name is required
validation.{模块名}_id_invalid=Invalid ID format
```

---

### 2.7 创建实体类

参考 [common-crud-rules](./common-crud-rules) 第一章节 "实体层(Entity)开发规范"

**关键点**:
- 继承 `GenericEntity<String>` 或 `GenericTreeSortSupportEntity<String>`(树形结构)
- 实现 `RecordCreationEntity` 和 `RecordModifierEntity` 接口
- 添加 `@Table`、`@Comment`、`@EnableEntityEvent` 注解
- 字段添加 `@Column`、`@Schema` 注解
- 使用 `@JsonCodec` 存储复杂对象
- 使用 `@EnumCodec` 处理枚举类型

**多模块注意事项**:
- API 模块：定义 DTO/VO 等对外接口实体，通常是简单的 POJO
- Manager 模块：定义数据库实体类（继承 `GenericEntity`）

---

### 2.8 创建服务类

参考 [common-crud-rules](./common-crud-rules) 第二章节 "服务层(Service)开发规范"

**响应式模式**:
- 继承 `GenericReactiveCrudService<Entity, String>`
- 所有方法返回 `Mono<T>` 或 `Flux<T>`

**阻塞式模式**:
- 继承 `GenericCrudService<Entity, String>`
- 返回普通 Java 对象、`List<T>` 或 `Optional<T>`

**多模块注意事项**:
- API 模块：定义服务接口（如需要跨服务调用）
- Manager 模块：实现服务接口和业务逻辑

---

### 2.9 创建控制器类

参考 [common-crud-rules](./common-crud-rules) 第三章节 "控制器层(Controller)开发规范"

**响应式模式**:
- 实现 `AssetsHolderCrudController<Entity, String>` 接口
- 所有方法返回 `Mono<T>` 或 `Flux<T>`

**阻塞式模式**:
- 实现 `BlockingAssetsHolderCrudController<Entity, String>` 接口
- 返回普通 Java 对象或 `List<T>`

**必需注解**:
- `@RestController`: 标记为REST控制器
- `@RequestMapping`: 定义路由前缀
- `@Authorize`: 启用权限控制
- `@Resource`: 定义资源信息
- `@Tag`: Swagger文档分组
- `@AssetsController`: 资产权限控制(可选)

**多模块注意事项**:
- 控制器仅在 Manager 模块中创建

---

### 2.10 创建资产类型(可选)

如果模块需要资产权限控制，创建资产类型定义:

#### 资产类型枚举

```java
package org.jetlinks.pro.{模块名}.assets;

import org.jetlinks.pro.assets.AssetType;

/**
 * {模块名称}资产类型
 */
public class {模块名首字母大写}AssetType {

    /**
     * {资产类型名称}
     */
    public static final AssetType {资产类型常量名} = AssetType.of("{资产类型ID}", "{资产类型名称}");

}
```

#### 资产注解

```java
package org.jetlinks.pro.{模块名}.assets;

import org.jetlinks.pro.assets.annotation.AssetsController;
import org.springframework.core.annotation.AliasFor;

import java.lang.annotation.*;

/**
 * {资产类型名称}资产注解
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@AssetsController(type = "{资产类型ID}")
public @interface {资产类型名称}Asset {

    @AliasFor(annotation = AssetsController.class)
    int assetIdIndex() default 0;

    @AliasFor(annotation = AssetsController.class)
    int assetObjectIndex() default -1;

    @AliasFor(annotation = AssetsController.class)
    String property() default "id";

    @AliasFor(annotation = AssetsController.class)
    boolean required() default false;

    @AliasFor(annotation = AssetsController.class)
    boolean autoBind() default false;

    @AliasFor(annotation = AssetsController.class)
    boolean autoUnbind() default false;

    @AliasFor(annotation = AssetsController.class)
    boolean ignore() default false;

    @AliasFor(annotation = AssetsController.class)
    boolean validate() default true;
}
```

#### 资产提供者

```java
package org.jetlinks.pro.{模块名}.assets;

import lombok.AllArgsConstructor;
import org.jetlinks.pro.assets.*;
import org.jetlinks.pro.{模块名}.entity.{实体名}Entity;
import org.jetlinks.pro.{模块名}.service.{实体名}Service;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.Collection;
import java.util.Collections;
import java.util.List;

/**
 * {资产类型名称}资产提供者
 */
@Component
@AllArgsConstructor
public class {资产类型名称}AssetSupplier implements AssetSupplier {

    private final {实体名}Service service;

    @Override
    public List<AssetType> getTypes() {
        return Collections.singletonList({模块名首字母大写}AssetType.{资产类型常量名});
    }

    @Override
    public Flux<DefaultAsset> getAssets(AssetType type, Collection<?> assetId) {
        return service
            .createQuery()
            .where()
            .in({实体名}Entity::getId, assetId)
            .fetch()
            .map(entity -> new DefaultAsset(
                entity.getId(),
                entity.getName(),
                type
            ));
    }

    @Override
    public Mono<Boolean> isSupport(AssetType type) {
        return Mono.just({模块名首字母大写}AssetType.{资产类型常量名}.isSameType(type));
    }
}
```

---

### 2.11 将模块添加到父 POM

编辑根目录的 `pom.xml`，在 `<modules>` 标签中添加:

**单模块示例**：
```xml
<modules>
    <!-- 其他模块 -->
    <module>modules/template-manager</module>
</modules>
```

**多模块示例**：
```xml
<modules>
    <!-- 其他模块 -->
    <module>modules/pms-report</module>  <!-- 只添加父模块 -->
</modules>
```

> **注意**: 
> - 单模块：直接添加模块目录，如 `modules/{模块名}-manager`
> - 多模块：只添加父模块目录，如 `modules/{项目简写}-{模块名}`，子模块（api、manager）由父模块的 POM 自动管理

---

## 三、模块开发检查清单

在创建模块后，使用以下检查清单确保模块完整性:

### 3.1 文件结构检查

- [ ] `pom.xml` 已创建，包含必要的依赖
- [ ] `Configuration` 类已创建，启用了 `@EnableEasyormRepository`
- [ ] `AutoConfiguration.imports` 文件已创建
- [ ] 国际化文件已创建 (messages_zh.properties, messages_en.properties)
- [ ] 目录结构完整 (entity, service, web, enums)

### 3.2 代码质量检查

- [ ] 所有类都有完整的 JavaDoc 注释
- [ ] 实体类包含所有必需注解 (@Table, @Column, @Schema)
- [ ] 服务类继承了正确的基类
  - [ ] 响应式: `GenericReactiveCrudService`
  - [ ] 阻塞式: `GenericCrudService`
- [ ] 控制器实现了正确的接口
  - [ ] 响应式: `AssetsHolderCrudController`
  - [ ] 阻塞式: `BlockingAssetsHolderCrudController`
- [ ] 权限注解配置完整 (@Resource, @Authorize, @QueryAction, @SaveAction)
- [ ] Swagger 文档注解完整 (@Tag, @Operation, @Parameter, @Schema)

### 3.3 功能检查

- [ ] 实体类字段验证规则正确 (@NotBlank, @Pattern 等)
- [ ] 枚举类实现了 `I18nEnumDict` 接口
- [ ] 服务层基础 CRUD 方法可用
- [ ] 控制器提供了完整的 RESTful 接口
- [ ] 如需要，资产类型和权限控制已配置

### 3.4 测试检查

- [ ] 服务层测试类已创建
- [ ] 控制器测试类已创建
- [ ] 基础 CRUD 操作测试通过

### 3.5 集成检查

- [ ] 模块已添加到父 POM 的 modules 列表
- [ ] 模块可以成功编译 (`mvn clean compile`)
- [ ] 模块可以成功打包 (`mvn clean package`)
- [ ] 在主应用中可以正常启动和使用

---

## 四、常见场景示例

### 4.1 创建简单的 CRUD 模块（单模块）

**场景**: 创建一个"配置模板"管理模块

```bash
# 1. 创建目录结构
mkdir -p modules/template-manager/src/main/java/org/jetlinks/pro/template/{entity,service,web,enums,configuration}
mkdir -p modules/template-manager/src/main/resources/{META-INF/spring,i18n/template-manager}
mkdir -p modules/template-manager/src/test/java/org/jetlinks/pro/template/{service,web}

# 2. 创建 pom.xml (使用上面的单模块基础模板)

# 3. 创建配置类
# src/main/java/org/jetlinks/pro/template/configuration/TemplateManagerConfiguration.java

# 4. 创建实体类
# src/main/java/org/jetlinks/pro/template/entity/ConfigTemplateEntity.java

# 5. 创建服务类
# src/main/java/org/jetlinks/pro/template/service/ConfigTemplateService.java

# 6. 创建控制器
# src/main/java/org/jetlinks/pro/template/web/ConfigTemplateController.java

# 7. 创建国际化文件
# src/main/resources/i18n/template-manager/messages_zh.properties
# src/main/resources/i18n/template-manager/messages_en.properties

# 8. 创建 AutoConfiguration.imports
# src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

### 4.2 创建支持跨服务调用的模块（多模块，推荐）

**场景**: 创建一个"任务调度"管理模块，需要被其他服务调用

**命名示例**：
- 父模块：`pms-scheduler`
- API 模块：`scheduler-api`
- Manager 模块：`scheduler-manager`

```bash
# 1. 创建父模块目录（{项目简写}-{模块名}）
mkdir -p modules/pms-scheduler

# 2. 创建父模块 POM
# modules/pms-scheduler/pom.xml
# artifactId: pms-scheduler
# modules: scheduler-api, scheduler-manager

# 3. 创建 API 模块目录结构
mkdir -p modules/pms-scheduler/scheduler-api/src/main/java/org/jetlinks/pro/scheduler/api/{entity,enums,command,query,event}
mkdir -p modules/pms-scheduler/scheduler-api/src/test/java/org/jetlinks/pro/scheduler/api

# 4. 创建 API 模块 POM
# modules/pms-scheduler/scheduler-api/pom.xml
# parent: pms-scheduler
# artifactId: scheduler-api

# 5. 在 API 模块创建接口定义
# - entity: 定义 DTO/VO (如 TaskDTO.java, TaskResultVO.java)
# - enums: 定义枚举 (如 TaskStatus.java)
# - command: 定义命令接口 (如 ExecuteTaskCommand.java)
# - query: 定义查询接口 (如 TaskQuery.java)
# - event: 定义事件 (如 TaskExecutedEvent.java)

# 6. 创建 Manager 模块目录结构
mkdir -p modules/pms-scheduler/scheduler-manager/src/main/java/org/jetlinks/pro/scheduler/{entity,service,web,command,configuration}
mkdir -p modules/pms-scheduler/scheduler-manager/src/main/resources/{META-INF/spring,i18n/scheduler-manager}
mkdir -p modules/pms-scheduler/scheduler-manager/src/test/java/org/jetlinks/pro/scheduler/{service,web}

# 7. 创建 Manager 模块 POM
# modules/pms-scheduler/scheduler-manager/pom.xml
# parent: pms-scheduler
# artifactId: scheduler-manager
# 依赖: scheduler-api

# 8. 在 Manager 模块创建实现
# - entity: 数据库实体类 (如 ScheduledTaskEntity.java)
# - service: 服务实现 (如 ScheduledTaskService.java)
# - web: 控制器 (如 ScheduledTaskController.java)
# - command: 命令实现 (如 ExecuteTaskCommandImpl.java)
# - configuration: 配置类 (如 SchedulerManagerConfiguration.java)

# 9. 创建配置文件
# modules/pms-scheduler/scheduler-manager/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
# 内容: org.jetlinks.pro.scheduler.configuration.SchedulerManagerConfiguration

# 10. 创建国际化文件
# modules/pms-scheduler/scheduler-manager/src/main/resources/i18n/scheduler-manager/messages_zh.properties
# modules/pms-scheduler/scheduler-manager/src/main/resources/i18n/scheduler-manager/messages_en.properties

# 11. 在根 POM 中添加模块
# 编辑 pom.xml，添加: <module>modules/pms-scheduler</module>
```

**跨服务调用示例**：

其他服务只需依赖 API 模块即可调用：

```xml
<!-- 在其他服务的 pom.xml 中添加 -->
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>scheduler-api</artifactId>
    <version>${project.version}</version>
</dependency>
```

然后在代码中使用：

```java
// 使用 API 模块定义的 DTO
import org.jetlinks.pro.scheduler.api.entity.TaskDTO;
import org.jetlinks.pro.scheduler.api.command.ExecuteTaskCommand;

// 执行任务
ExecuteTaskCommand command = new ExecuteTaskCommand();
command.setTaskId(taskId);
command.setParameters(params);
// ...调用服务
```

### 4.3 创建带资产权限的模块

**场景**: 创建一个"数据源"管理模块，需要资产权限控制

```bash
# 基础结构同上，额外创建:

# 1. 资产类型目录
mkdir -p modules/datasource-manager/src/main/java/org/jetlinks/pro/datasource/assets

# 2. 创建资产类型枚举
# src/main/java/org/jetlinks/pro/datasource/assets/DatasourceAssetType.java

# 3. 创建资产注解
# src/main/java/org/jetlinks/pro/datasource/assets/DatasourceAsset.java

# 4. 创建资产提供者
# src/main/java/org/jetlinks/pro/datasource/assets/DatasourceAssetSupplier.java

# 5. 控制器使用资产注解
# @DatasourceAsset 或 @AssetsController(type = "datasource")
```

### 4.4 创建树形结构模块

**场景**: 创建一个"组织架构"管理模块，需要树形结构

```java
// 1. 实体类继承树形基类
public class OrganizationEntity extends GenericTreeSortSupportEntity<String>
    implements RecordCreationEntity {

    @Schema(description = "子节点")
    private List<OrganizationEntity> children;

    // 其他字段...
}

// 2. 服务类继承树形服务
@Service
public class OrganizationService extends GenericReactiveTreeSupportCrudService<OrganizationEntity, String> {

    @Override
    public void setChildren(OrganizationEntity entity, List<OrganizationEntity> children) {
        entity.setChildren(children);
    }

    @Override
    public IDGenerator<String> getIDGenerator() {
        return IDGenerator.SNOW_FLAKE;
    }
}

// 3. 控制器添加树形查询接口
@GetMapping("/_tree")
@QueryAction
@Operation(summary = "查询树形结构")
public Flux<OrganizationEntity> queryTree(@Parameter(hidden = true) QueryParamEntity query) {
    return service
        .createQuery()
        .setParam(query)
        .fetch()
        .collectList()
        .flatMapMany(all -> Flux.fromIterable(
            TreeSupportEntity.list2tree(all, OrganizationEntity::setChildren)
        ));
}
```

---

## 五、依赖选择指南

### 5.1 必需依赖(所有模块都需要)

```xml
<!-- 通用组件 -->
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>common-component</artifactId>
    <version>${project.version}</version>
</dependency>

<!-- HSWeb Starter -->
<dependency>
    <groupId>org.hswebframework.web</groupId>
    <artifactId>hsweb-starter</artifactId>
    <version>${hsweb.framework.version}</version>
</dependency>

<!-- HSWeb 权限 API -->
<dependency>
    <groupId>org.hswebframework.web</groupId>
    <artifactId>hsweb-authorization-api</artifactId>
    <version>${hsweb.framework.version}</version>
</dependency>

<!-- EasyORM -->
<dependency>
    <groupId>org.hswebframework</groupId>
    <artifactId>hsweb-easy-orm-rdb</artifactId>
</dependency>

<!-- R2DBC H2 -->
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-h2</artifactId>
</dependency>
```

### 5.2 按场景选择依赖

#### 需要资产权限控制

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>assets-component</artifactId>
    <version>${project.version}</version>
</dependency>
```

#### 需要时序数据存储

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>timeseries-component</artifactId>
    <version>${project.version}</version>
</dependency>
```

#### 需要设备相关功能

```xml
<dependency>
    <groupId>org.jetlinks</groupId>
    <artifactId>jetlinks-core</artifactId>
    <version>${jetlinks.version}</version>
</dependency>

<dependency>
    <groupId>org.jetlinks</groupId>
    <artifactId>jetlinks-supports</artifactId>
    <version>${jetlinks.version}</version>
</dependency>

<dependency>
    <groupId>org.jetlinks.sdk</groupId>
    <artifactId>jetlinks-sdk-api</artifactId>
</dependency>
```

#### 需要搜索功能

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>elasticsearch-core</artifactId>
    <version>${project.version}</version>
    <optional>true</optional>
</dependency>
```

#### 需要实体关系管理

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>relation-component</artifactId>
    <version>${project.version}</version>
</dependency>
```

#### 需要通知功能

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>notify-core</artifactId>
    <optional>true</optional>
</dependency>

<!-- 根据需要添加具体通知类型 -->
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>notify-email</artifactId>
    <version>${project.version}</version>
</dependency>

<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>notify-sms</artifactId>
    <version>${project.version}</version>
</dependency>
```

#### 需要规则引擎

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>rule-engine-component</artifactId>
    <version>${project.version}</version>
    <optional>true</optional>
</dependency>
```

#### 需要脚本执行

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>script-component</artifactId>
    <version>${project.version}</version>
</dependency>
```

#### 需要 OpenAPI 支持

```xml
<dependency>
    <groupId>org.jetlinks.pro</groupId>
    <artifactId>api-component</artifactId>
    <version>${project.version}</version>
</dependency>
```

#### 需要数据字典

```xml
<dependency>
    <groupId>org.hswebframework.web</groupId>
    <artifactId>hsweb-system-dictionary</artifactId>
</dependency>
```

---

## 六、最佳实践

### 6.1 模块命名规范

#### 单模块命名
- **目录名**: `{模块名}-manager`，如 `template-manager`
- **artifactId**: 与目录名一致
- **包名**: 使用驼峰命名，如 `org.jetlinks.pro.template`
- **类名**: 首字母大写的驼峰命名，如 `TemplateManagerConfiguration`

#### 多模块命名（推荐）
- **父模块目录**: `{项目简写}-{模块名}`，如 `pms-report`
- **父模块 artifactId**: 与目录名一致，如 `pms-report`
- **API 模块**:
  - 目录名: `{模块名}-api`，如 `report-api`
  - artifactId: 与目录名一致
  - 包名: `org.jetlinks.pro.{模块名}.api`，如 `org.jetlinks.pro.report.api`
- **Manager 模块**:
  - 目录名: `{模块名}-manager`，如 `report-manager`
  - artifactId: 与目录名一致
  - 包名: `org.jetlinks.pro.{模块名}`，如 `org.jetlinks.pro.report`
  - 类名: 如 `ReportManagerConfiguration`

#### 项目简写示例
- `pms`: Platform Management System（平台管理系统）
- `dms`: Device Management System（设备管理系统）
- `ams`: Asset Management System（资产管理系统）
- `wms`: Workflow Management System（工作流管理系统）

### 6.2 代码组织

- **entity**: 只包含实体类，不包含业务逻辑
- **service**: 包含业务逻辑，不直接处理HTTP请求
- **web**: 只包含控制器，处理HTTP请求，调用服务层
- **enums**: 包含枚举类，实现 `I18nEnumDict` 接口
- **assets**: 包含资产类型定义和提供者
- **configuration**: 包含配置类和属性类

### 6.3 注释规范

```java
/**
 * {类描述} - 详细说明该类的用途
 * <p>
 * 如果有特殊说明，可以在这里补充
 * </p>
 *
 * @author {作者名}
 * @since {版本号}
 */
@Service
@Slf4j
public class ExampleService {

    /**
     * {方法描述} - 详细说明方法的功能
     *
     * @param id 参数说明
     * @return 返回值说明
     */
    public Mono<Entity> findById(String id) {
        // ...
    }
}
```

### 6.4 异常处理

- 使用 `BusinessException` 表示业务异常
- 使用 `NotFoundException` 表示资源不存在
- 使用 `ValidationException` 表示验证失败
- 响应式代码使用 `onErrorResume`、`onErrorReturn` 处理异常
- 阻塞式代码使用 try-catch 或抛出异常

### 6.5 日志记录

```java
// 使用 @Slf4j 注解
@Service
@Slf4j
public class ExampleService {

    public Mono<Entity> save(Entity entity) {
        return Mono.just(entity)
            .doOnNext(e -> log.info("保存数据: {}", e.getId()))
            .flatMap(e -> repository.save(e))
            .doOnSuccess(e -> log.info("保存成功: {}", e.getId()))
            .doOnError(err -> log.error("保存失败", err));
    }
}
```

### 6.6 性能优化

- 为常用查询字段添加数据库索引
- 使用 `select` 指定需要的字段，避免查询所有字段
- 批量操作使用 `Flux.buffer()` 控制批次大小
- 使用缓存减少数据库访问 (`@Cacheable`)

---

## 七、常见问题

### Q1: 模块编译失败，提示找不到某些类？

**A**: 检查 pom.xml 依赖是否完整，特别是:
- `common-component`: 必需
- `hsweb-starter`: 必需
- 相关业务组件是否已添加

### Q2: 模块启动后接口无法访问？

**A**: 检查:
1. `AutoConfiguration.imports` 文件是否创建
2. Configuration 类上的 `@EnableEasyormRepository` 注解是否正确
3. 控制器上的 `@RestController` 和 `@RequestMapping` 注解是否正确

### Q3: 权限控制不生效？

**A**: 检查:
1. 控制器类是否添加了 `@Authorize` 注解
2. `@Resource` 注解的 id 和 name 是否正确
3. 方法是否添加了 `@QueryAction`、`@SaveAction` 等权限注解
4. 用户是否被授予了相应的权限

### Q4: 资产权限不生效？

**A**: 检查:
1. `@AssetsController` 注解是否添加
2. AssetSupplier 是否实现并注册为 Spring Bean
3. 资产类型定义是否正确
4. 用户是否被授予了相应的资产权限

### Q5: 响应式编程遇到阻塞问题？

**A**: 
- 避免在 Reactor 链中使用 `.block()`
- 使用 `Mono.fromCallable()` 包装阻塞式代码
- 考虑使用阻塞式编程模式 (Spring MVC)

### Q6: 如何在响应式和阻塞式之间选择?

**A**: 参考以下建议:
- **高并发、IO密集型** → 使用响应式
- **传统业务、低并发** → 使用阻塞式
- **团队不熟悉响应式** → 使用阻塞式
- **已有大量阻塞式依赖** → 使用阻塞式

---

## 八、示例模块参考

### 8.1 简单 CRUD 模块
- `modules/datasource-manager`: 数据源管理
- `modules/logging-manager`: 日志管理

### 8.2 复杂业务模块
- `modules/device-manager`: 设备管理(资产权限、时序数据、搜索)
- `modules/notify-manager`: 通知管理(多种通知类型)
- `modules/rule-engine-manager`: 规则引擎(流程编排、脚本)

### 8.3 认证授权模块
- `modules/authentication-manager`: 认证管理(OAuth2、单点登录)

---

## 总结

创建 JetLinks 业务模块的核心步骤:

### 通用步骤：

1. ✅ **明确需求**: 模块名称、项目简写、技术选型、业务特性、是否需要跨服务调用
2. ✅ **选择模块结构**: 
   - **单模块**: 简单业务，不需要跨服务调用（`{模块名}-manager`）
   - **多模块（推荐）**: 需要跨服务调用或模块较复杂（`{项目简写}-{模块名}/{模块名}-api/{模块名}-manager`）
3. ✅ **创建 pom.xml**: 
   - 单模块：创建一个 POM（artifactId: `{模块名}-manager`）
   - 多模块：创建三个 POM
     - 父 POM（artifactId: `{项目简写}-{模块名}`，如 `pms-report`）
     - API POM（artifactId: `{模块名}-api`，如 `report-api`）
     - Manager POM（artifactId: `{模块名}-manager`，如 `report-manager`）
4. ✅ **创建目录结构**: 
   - 单模块：entity、service、web、enums、configuration
   - 多模块：
     - API 模块（entity、enums、command、query、event、constants）
     - Manager 模块（entity、service、web、command、configuration）
5. ✅ **创建配置类**: Configuration + AutoConfiguration.imports（在 Manager 模块）
6. ✅ **创建国际化文件**: messages_zh.properties + messages_en.properties（在 Manager 模块）
7. ✅ **创建实体类**: 
   - API 模块：DTO/VO（简单 POJO）
   - Manager 模块：数据库实体（继承基类、添加注解、定义字段）
8. ✅ **创建服务类**: 
   - API 模块：服务接口（如需跨服务调用）
   - Manager 模块：服务实现（继承 CRUD 基类、实现业务逻辑）
9. ✅ **创建控制器**: 实现 CRUD 接口、添加权限注解（在 Manager 模块）
10. ✅ **创建资产类型**: 如需要资产权限控制（在 Manager 模块）
11. ✅ **添加到父 POM**: 在根 POM 中添加模块引用
12. ✅ **测试验证**: 编译、打包、启动、测试

### 多模块结构优势：

- ✅ **解耦合**: API 和实现分离，其他服务只依赖接口
- ✅ **可扩展**: 便于版本管理和演进
- ✅ **轻依赖**: API 模块依赖少，不会引入过多传递依赖
- ✅ **跨服务**: 天然支持微服务架构中的服务间调用
- ✅ **职责清晰**: API 定义契约，Manager 提供实现

### 多模块命名规范总结：

**命名结构**：
```
modules/{项目简写}-{模块名}/        # 父模块（如 pms-report）
  ├── {模块名}-api/                 # API 模块（如 report-api）
  └── {模块名}-manager/             # Manager 实现模块（如 report-manager）
```

**示例**：
```
modules/pms-report/                # 报表模块父目录
  ├── report-api/                  # 报表 API 接口定义
  └── report-manager/              # 报表管理实现
```

### 建议：

> 🎯 **新建模块默认使用多模块结构**，即使暂时不需要跨服务调用，这样可以为未来扩展留下空间，避免后期重构的成本。

> 📝 **命名规范统一使用 `-manager`**，实现模块统一命名为 `{模块名}-manager`，而不是 `-service`，保持与单模块命名的一致性。

遵循本指南可以快速、规范地创建高质量的业务模块。

详细的代码规范请参考: [common-crud-rules](./common-crud-rules)

