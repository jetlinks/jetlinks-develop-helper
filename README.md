# JetLinks 开发辅助工具 - AI 开发规则库

本目录包含用于辅助 AI 快速、规范地生成 JetLinks 项目代码的完整规则集。

---

## 📚 目录

- [快速开始](#快速开始)
- [规则文件说明](#规则文件说明)
- [AI 使用指南](#ai-使用指南)
- [使用场景决策树](#使用场景决策树)
- [完整示例](#完整示例)

---

## 🚀 快速开始

### 对于 AI 助手

当用户请求创建或修改 JetLinks 代码时：

1. **首先判断任务类型**：
    - 创建新模块 → 使用 [module-creation-rules](ai/module-creation-rules.md)
    - 在现有模块中添加 CRUD 功能 → 使用 [common-crud-rules](ai/common-crud-rules.md)
    - 两者都需要 → 先使用 module-creation-rules，再参考 common-crud-rules

2. **读取相应的规则文件**：
   ```
   请使用 read_file 工具读取完整规则，而不是依赖记忆
   ```

3. **严格遵循规则**：
    - 使用规则中的代码模板
    - 保持命名规范一致
    - 包含所有必需的注解和配置

4. **生成完整代码**：
    - 不要只生成片段，要生成完整的类
    - 包含必要的导入语句
    - 添加完整的注释和文档

---

## 📖 规则文件说明

### 1. 模块创建规则 - [module-creation-rules](ai/module-creation-rules.md)

**用途**: 从零创建一个新的业务模块（完整的 Maven 模块）

**包含内容**:

- ✅ 完整的 Maven pom.xml 配置模板
- ✅ 目录结构规范和创建步骤
- ✅ 必需依赖和可选依赖说明（含详细表格）
- ✅ Configuration 配置类模板
- ✅ AutoConfiguration.imports 配置
- ✅ 国际化文件模板
- ✅ 资产类型定义模板（可选）
- ✅ 模块开发检查清单
- ✅ 常见场景示例（简单 CRUD、资产权限、树形结构）
- ✅ 依赖选择指南（8种常见场景）

**适用场景**:

- ✨ 创建新的业务模块（如：template-manager, workflow-manager）
- ✨ 需要完整的 Maven 模块结构
- ✨ 需要独立的 pom.xml 和配置

**何时使用**:

```
用户说："创建一个XXX管理模块"
用户说："新建一个模块用于XXX"
用户说："我需要一个新的业务模块"
```

---

### 2. 实时数据订阅规则 - [realtime-subscription-rules](ai/realtime-subscription-rules.md)

**用途**: 使用 @Subscribe 注解和 EventBus 实现实时数据订阅

**包含内容**:

- ✅ Subscribe 注解详解和使用方法
- ✅ Topic 规则和通配符使用
- ✅ 订阅设备消息的各种方式
- ✅ 常见应用场景(4个核心示例)
  - 订阅设备属性上报并更新 Redis
  - 订阅设备上下线并发送通知
  - 订阅设备事件并处理告警
  - 使用 Buffer 处理耗时操作(持久化缓冲)
- ✅ 订阅特性详解(本地、集群、共享订阅等)
- ✅ 性能优化建议
- ✅ 常见问题解答(5个核心问题)
- ✅ 快速参考表

**适用场景**:

- ✨ 订阅设备实时消息(属性上报、事件、上下线等)
- ✨ 实现数据实时同步到 Redis、数据库等
- ✨ 处理设备告警和通知
- ✨ 实时数据流处理和转发
- ✨ 集群环境下的消息订阅

**何时使用**:

```
用户说："订阅设备属性上报数据"
用户说："监听设备上下线消息"
用户说："实时处理设备事件"
用户说："将设备数据同步到Redis"
用户说："如何使用@Subscribe注解"
```

---

### 3. 事件驱动开发规范 - [event-driven-rules](ai/event-driven-rules.md)

**用途**: 使用 Spring Event 处理实体增删改查事件实现业务解耦

**包含内容**:

- ✅ 通用CRUD事件监听(响应式和阻塞式)
- ✅ 完整的事件类型说明表
- ✅ event.async() 响应式操作组合
- ✅ @TransactionalEventListener 事务事件
- ✅ 集群事件监听(ClusterEntityEventListener)
- ✅ 实战示例(4个核心场景)
  - 设备激活时同步资产
  - 实体保存时发布到EventBus
  - 用户Token变更通知
  - 获取修改前后的值对比
- ✅ 开启实体事件(@EnableEntityEvent)
- ✅ 注意事项和最佳实践

**适用场景**:

- ✨ 监听实体的增删改查操作
- ✨ 数据变更后触发业务逻辑
- ✨ 实现功能解耦和异步处理
- ✨ 数据同步、日志记录、消息通知
- ✨ 集群环境下的事件同步

**何时使用**:

```
用户说："监听实体保存事件"
用户说："数据修改后发送通知"
用户说："如何使用@EventListener"
用户说："实体删除时清理关联数据"
用户说："使用事件驱动实现业务解耦"
```

---

### 4. 通用 CRUD 规则 - [common-crud-rules](ai/common-crud-rules.md)

**用途**: 在现有模块中创建或修改 CRUD 功能相关代码

**包含内容**:

- ✅ Entity（实体类）开发规范和模板
    - 基础实体、树形实体
    - 枚举类型定义
    - 所有注解的详细说明
- ✅ Service（服务层）开发规范和模板
    - 响应式服务（GenericReactiveCrudService）
    - 阻塞式服务（GenericCrudService）
    - 树形服务、查询示例
- ✅ Controller（控制器层）开发规范和模板
    - 响应式控制器（AssetsHolderCrudController）
    - 阻塞式控制器（BlockingAssetsHolderCrudController）
    - 权限注解、资产权限控制
- ✅ 响应式 vs 阻塞式编程模式对比
- ✅ 权限控制最佳实践
- ✅ 查询模式示例（基础查询、分页、关联查询、分组统计）
- ✅ 响应式编程规范（Mono/Flux 操作符）
- ✅ 国际化支持
- ✅ 异常处理规范
- ✅ 测试规范
- ✅ 代码生成指南（供 AI 使用）

**适用场景**:

- ✨ 在现有模块中添加新的实体和 CRUD 功能
- ✨ 修改现有的 Entity/Service/Controller
- ✨ 理解 JetLinks CRUD 代码规范
- ✨ 学习响应式或阻塞式编程模式

**何时使用**:

```
用户说："在XXX模块中添加一个XXX实体"
用户说："创建一个XXX的增删改查功能"
用户说："修改XXXController添加一个接口"
用户说："为XXX添加权限控制"
```

---

## 🤖 AI 使用指南

### 基本原则

1. **始终读取完整规则**
   ```
   不要依赖记忆，使用 read_file 读取完整的规则文件
   规则文件包含最新的模板和最佳实践
   ```

2. **明确用户需求**
   ```
   - 模块名称是什么？
   - 使用响应式还是阻塞式编程？（默认：响应式）
   - 需要哪些实体和字段？
   - 是否需要资产权限控制？
   - 是否需要树形结构？
   ```

3. **生成完整代码**
   ```
   - 包含完整的包声明和导入语句
   - 包含所有必需的注解
   - 添加完整的 JavaDoc 注释
   - 提供 Swagger 文档注解
   ```

4. **遵循命名规范**
   ```
   - 模块名: kebab-case (template-manager)
   - 包名: camelCase (org.jetlinks.pro.template)
   - 类名: PascalCase (TemplateManagerConfiguration)
   - 变量名: camelCase (templateService)
   ```

5. **使用检查清单**
   ```
   代码生成后，参考规则文件中的检查清单验证完整性
   ```

### 工作流程

```
┌─────────────────────────────────────────────────┐
│  1. 接收用户需求                                  │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  2. 判断任务类型                                  │
│     ├─ 创建新模块？→ module-creation-rules      │
│     ├─ 添加CRUD？  → common-crud-rules          │
│     └─ 两者都需要？→ 两个规则都使用              │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  3. 使用 read_file 读取完整规则                   │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  4. 收集详细信息                                  │
│     - 模块名称、实体名称、字段列表                │
│     - 编程模式（响应式/阻塞式）                   │
│     - 权限和资产要求                              │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  5. 根据模板生成代码                              │
│     - 使用规则文件中的代码模板                    │
│     - 替换占位符                                  │
│     - 保持完整性                                  │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  6. 验证和检查                                    │
│     - 使用检查清单验证                            │
│     - 确保所有注解完整                            │
│     - 提供使用说明                                │
└─────────────────────────────────────────────────┘
```

---

## 🌳 使用场景决策树

```
用户的请求
    │
    ├─ "创建一个XXX管理模块"
    │   │
    │   ├─ 需要完整的模块结构（pom.xml、配置类等）
    │   │   └─→ 使用 module-creation-rules
    │   │       └─→ 生成：pom.xml, Configuration, Entity, Service, Controller, 国际化文件
    │   │
    │   └─ 只需要 Entity/Service/Controller
    │       └─→ 使用 common-crud-rules
    │           └─→ 生成：Entity, Service, Controller
    │
    ├─ "订阅设备XXX消息" / "监听设备XXX"
    │   └─→ 使用 realtime-subscription-rules
    │       ├─ 查看常见 Topic 速查表
    │       ├─ 选择合适的订阅方式（@Subscribe 注解 或 EventBus）
    │       └─→ 参考相应场景的完整示例
    │
    ├─ "将设备数据同步到Redis/数据库"
    │   └─→ 使用 realtime-subscription-rules
    │       └─→ 参考"场景1: 订阅设备属性上报并更新Redis"
    │
    ├─ "实时处理设备事件/告警"
    │   └─→ 使用 realtime-subscription-rules
    │       └─→ 参考"场景3: 订阅设备事件并处理告警"
    │
    ├─ "监听实体增删改查" / "数据变更后处理"
    │   └─→ 使用 event-driven-rules
    │       ├─ 查看完整事件类型说明表
    │       ├─ 选择响应式或阻塞式环境
    │       └─→ 参考实战示例
    │
    ├─ "实体保存/删除/修改后发送通知"
    │   └─→ 使用 event-driven-rules
    │       └─→ 参考"实体保存时发布到EventBus"示例
    │
    ├─ "使用事件驱动实现业务解耦"
    │   └─→ 使用 event-driven-rules
    │       └─→ 查看通用CRUD事件监听章节
    │
    ├─ "在XXX模块中添加XXX功能"
    │   └─→ 使用 common-crud-rules
    │       └─→ 根据需求生成相应的类
    │
    ├─ "修改XXXController添加接口"
    │   └─→ 使用 common-crud-rules
    │       └─→ 参考第三章"控制器层开发规范"
    │
    ├─ "创建一个XXX实体类"
    │   └─→ 使用 common-crud-rules
    │       └─→ 参考第一章"实体层开发规范"
    │
    ├─ "需要添加资产权限控制"
    │   └─→ 同时参考两个规则
    │       ├─ module-creation-rules: 2.9 创建资产类型
    │       └─ common-crud-rules: 3.3 关联资产权限控制
    │
    └─ "如何在JetLinks中实现XXX"
        └─→ 先查看 common-crud-rules 相关章节
            └─→ 如需模块级配置，参考 module-creation-rules
            └─→ 如需实时数据处理，参考 realtime-subscription-rules
```

---

## 📝 完整示例

### 示例 1: 创建完整的新模块

**用户请求**:

```markdown
创建一个配置模板管理模块(template-manager)，
用于管理各种配置模板，使用响应式编程。
```

**AI 处理步骤**:

1. **读取规则**:
   ```
   read_file: ai/module-creation-rules (获取完整的模块创建指南)
   read_file: ai/common-crud-rules (获取 Entity/Service/Controller 规范)
   ```

2. **收集信息**:
   ```
   - 模块名: template-manager
   - 包名: org.jetlinks.pro.template
   - 编程模式: 响应式(WebFlux)
   - 核心实体: ConfigTemplateEntity
   - 主要字段: name, type, content, state
   ```

3. **生成文件**:
   ```
   ✅ pom.xml (使用 module-creation-rules 第2.2节模板)
   ✅ TemplateManagerConfiguration.java (第2.4节)
   ✅ AutoConfiguration.imports (第2.4节)
   ✅ messages_zh.properties & messages_en.properties (第2.5节)
   ✅ ConfigTemplateEntity.java (common-crud-rules 第1.1节)
   ✅ TemplateTypeEnum.java (common-crud-rules 第1.3节)
   ✅ ConfigTemplateService.java (common-crud-rules 第2.1节)
   ✅ ConfigTemplateController.java (common-crud-rules 第3.1节)
   ```

4. **验证**:
   ```
   使用 module-creation-rules 第3章"模块开发检查清单"验证
   ```

---

### 示例 2: 在现有模块中添加 CRUD 功能

**用户请求**:

```markdown
在 device-manager 模块中添加设备标签(DeviceTag)的管理功能，
包括增删改查接口。
```

**AI 处理步骤**:

1. **读取规则**:
   ```
   read_file: ai/common-crud-rules (获取完整的 CRUD 开发规范)
   ```

2. **收集信息**:
   ```
   - 模块: device-manager (已存在)
   - 实体名: DeviceTagEntity
   - 包名: org.jetlinks.pro.device
   - 编程模式: 响应式(根据现有模块)
   - 字段: name, color, deviceId 等
   ```

3. **生成文件**:
   ```
   ✅ DeviceTagEntity.java (第1.1节模板)
   ✅ DeviceTagService.java (第2.1节模板)
   ✅ DeviceTagController.java (第3.1节模板)
   ✅ 更新 messages_zh.properties (第7.1节)
   ```

---

### 示例 3: 添加资产权限控制

**用户请求**:

```markdown
为 template-manager 模块添加资产权限控制。
```

**AI 处理步骤**:

1. **读取规则**:
   ```
   read_file: ai/module-creation-rules (第2.9节 - 创建资产类型)
   read_file: ai/common-crud-rules (第3.3节 - 关联资产权限控制)
   ```

2. **生成文件**:
   ```
   ✅ TemplateAssetType.java (资产类型枚举)
   ✅ TemplateAsset.java (资产注解)
   ✅ TemplateAssetSupplier.java (资产提供者)
   ✅ 更新 pom.xml (添加 assets-component 依赖)
   ✅ 更新 ConfigTemplateController (添加 @TemplateAsset 注解)
   ```

---

### 示例 4: 创建树形结构模块

**用户请求**:

```markdown
创建一个组织架构管理模块(organization-manager)，
需要支持树形结构。
```

**AI 处理步骤**:

1. **读取规则**:
   ```
   read_file: ai/module-creation-rules (完整模块创建指南)
   read_file: ai/common-crud-rules (第1.2节 - 树形实体，第2.2节 - 树形服务)
   ```

2. **关键点**:
   ```
   - Entity 继承 GenericTreeSortSupportEntity
   - Service 继承 GenericReactiveTreeSupportCrudService
   - Controller 添加 /_tree 接口
   ```

---

## 🎯 AI 快速参考

### 常用代码模板快速定位

| 需求              | 规则文件                     | 章节    |
|-----------------|--------------------------|-------|
| Maven pom.xml   | module-creation-rules    | 2.2   |
| Configuration 类 | module-creation-rules    | 2.4   |
| 基础实体类           | common-crud-rules        | 1.1   |
| 响应式服务           | common-crud-rules        | 2.1   |
| 阻塞式服务           | common-crud-rules        | 2.A.1 |
| 响应式控制器          | common-crud-rules        | 3.1   |
| 阻塞式控制器          | common-crud-rules        | 3.A.1 |
| 资产类型定义          | module-creation-rules    | 2.9   |
| 权限控制            | common-crud-rules        | 4     |
| **订阅设备消息**      | **realtime-subscription-rules** | **全文** |
| **@Subscribe 注解**  | **realtime-subscription-rules** | **全文** |
| **实时数据处理**      | **realtime-subscription-rules** | **场景示例** |
| **实体事件监听**      | **event-driven-rules** | **一、二** |
| **@EventListener** | **event-driven-rules** | **一** |
| **event.async()** | **event-driven-rules** | **一、六** |
| **集群事件监听**      | **event-driven-rules** | **四** |

### 必需注解检查清单

```java
// Entity
@Table(name = "表名")
@Comment("表注释")
@EnableEntityEvent  // 开启实体事件
@Column
@Schema

// Service
@Service
@Slf4j

// Controller
@RestController
@RequestMapping("/路径")
@Authorize
@Resource(id = "资源ID", name = "资源名称")
@Tag(name = "API标签")
@AssetsController(type = "资产类型")  // 如需要
@QueryAction /@SaveAction /@DeleteAction
@Operation(summary = "接口说明")

// Event Handler (响应式环境)
@Component
@Slf4j
@EventListener
// 使用 event.async() 包装响应式操作

// Event Handler (阻塞式环境)
@Component
@Slf4j
@EventListener 或 @TransactionalEventListener
```

---

## 💡 提示

- **对于 AI**: 请始终完整读取规则文件，不要依赖记忆或假设
- **对于开发者**: 这些规则同样适用于人工开发，可以作为开发规范参考
- **保持更新**: 规则文件会持续更新，请关注最新版本

---

## 📞 反馈

如发现规则不完善或需要补充的内容，请及时更新规则文件。
