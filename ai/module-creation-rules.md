# JetLinks 业务模块创建指南

本指南提供了在 JetLinks 项目中创建新业务模块的完整流程和规范。

## 一、模块结构概述

JetLinks 项目采用模块化架构，所有业务模块位于 `modules/` 目录下：

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

---

## 二、创建模块步骤

### 2.1 明确模块需求

在创建模块前，需要明确以下信息：

#### 基础信息
- **模块名称**: 如 `device`, `notify`, `authentication`（使用短横线命名，如 `rule-engine`）
- **模块描述**: 简要说明模块用途
- **包名**: 通常为 `org.jetlinks.pro.{模块名}`
- **artifactId**: 通常为 `{模块名}-manager`

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

---

### 2.2 创建 pom.xml

#### 基础模板

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

### 2.3 创建目录结构

```bash
# 创建主要目录
mkdir -p modules/{模块名}-manager/src/main/java/org/jetlinks/pro/{模块名}/{entity,service,web,enums}
mkdir -p modules/{模块名}-manager/src/main/resources/META-INF/spring
mkdir -p modules/{模块名}-manager/src/main/resources/i18n/{模块名}-manager
mkdir -p modules/{模块名}-manager/src/test/java/org/jetlinks/pro/{模块名}/{service,web}
```

---

### 2.4 创建配置类

#### AutoConfiguration.imports

创建文件: `src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

```
org.jetlinks.pro.{模块名}.configuration.{模块名首字母大写}ManagerConfiguration
```

#### Configuration 类

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

---

### 2.5 创建国际化文件

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

### 2.6 创建实体类

参考 [common-crud-rules](./common-crud-rules) 第一章节 "实体层(Entity)开发规范"

**关键点**:
- 继承 `GenericEntity<String>` 或 `GenericTreeSortSupportEntity<String>`(树形结构)
- 实现 `RecordCreationEntity` 和 `RecordModifierEntity` 接口
- 添加 `@Table`、`@Comment`、`@EnableEntityEvent` 注解
- 字段添加 `@Column`、`@Schema` 注解
- 使用 `@JsonCodec` 存储复杂对象
- 使用 `@EnumCodec` 处理枚举类型

---

### 2.7 创建服务类

参考 [common-crud-rules](./common-crud-rules) 第二章节 "服务层(Service)开发规范"

**响应式模式**:
- 继承 `GenericReactiveCrudService<Entity, String>`
- 所有方法返回 `Mono<T>` 或 `Flux<T>`

**阻塞式模式**:
- 继承 `GenericCrudService<Entity, String>`
- 返回普通 Java 对象、`List<T>` 或 `Optional<T>`

---

### 2.8 创建控制器类

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

---

### 2.9 创建资产类型(可选)

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

### 2.10 将模块添加到父 POM

编辑根目录的 `pom.xml`，在 `<modules>` 标签中添加:

```xml
<modules>
    <!-- 其他模块 -->
    <module>modules/{模块名}-manager</module>
</modules>
```

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

### 4.1 创建简单的 CRUD 模块

**场景**: 创建一个"配置模板"管理模块

```bash
# 1. 创建目录结构
mkdir -p modules/template-manager/src/main/java/org/jetlinks/pro/template/{entity,service,web,enums,configuration}
mkdir -p modules/template-manager/src/main/resources/{META-INF/spring,i18n/template-manager}
mkdir -p modules/template-manager/src/test/java/org/jetlinks/pro/template/{service,web}

# 2. 创建 pom.xml (使用上面的基础模板)

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

### 4.2 创建带资产权限的模块

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

### 4.3 创建树形结构模块

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

- **目录名**: 使用短横线分隔，如 `device-manager`, `rule-engine-manager`
- **artifactId**: 与目录名一致
- **包名**: 使用驼峰命名，如 `org.jetlinks.pro.device`, `org.jetlinks.pro.ruleengine`
- **类名**: 首字母大写的驼峰命名，如 `DeviceManagerConfiguration`

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

1. ✅ **明确需求**: 模块名称、技术选型、业务特性
2. ✅ **创建 pom.xml**: 添加必需和可选依赖
3. ✅ **创建目录结构**: entity、service、web、enums、configuration
4. ✅ **创建配置类**: Configuration + AutoConfiguration.imports
5. ✅ **创建国际化文件**: messages_zh.properties + messages_en.properties
6. ✅ **创建实体类**: 继承基类、添加注解、定义字段
7. ✅ **创建服务类**: 继承 CRUD 基类、实现业务逻辑
8. ✅ **创建控制器**: 实现 CRUD 接口、添加权限注解
9. ✅ **创建资产类型**: 如需要资产权限控制
10. ✅ **测试验证**: 编译、打包、启动、测试

遵循本指南可以快速、规范地创建高质量的业务模块。

详细的代码规范请参考: [common-crud-rules](./common-crud-rules)

