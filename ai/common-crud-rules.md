# JetLinks 项目开发规范 - AI辅助规则

## 项目概述
JetLinks是基于Spring Boot的物联网平台，使用Reactive响应式编程模型，采用模块化架构设计。

## 核心技术栈
- Spring Boot + WebFlux (响应式编程) / Spring MVC (阻塞式编程)
- EasyORM (响应式ORM框架 / 阻塞式ORM)
- HSWebFramework (权限控制框架)
- Reactor (Mono/Flux) - 响应式编程
- PostgreSQL/MySQL (关系型数据库)

## 编程模式说明
JetLinks 支持两种编程模式：
1. **响应式编程模式(WebFlux)**: 适用于高并发、IO密集型场景，使用 Mono/Flux，非阻塞
2. **阻塞式编程模式(Spring MVC)**: 适用于传统业务场景，使用普通Java对象，代码简单直观

本规范同时包含两种模式的开发指南。

---

## 一、实体层(Entity)开发规范

### 1.1 基础实体类模板

```java
package org.jetlinks.pro.{模块名}.entity;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;
import lombok.Setter;
import org.hswebframework.ezorm.rdb.mapping.annotation.*;
import org.hswebframework.web.api.crud.entity.GenericEntity;
import org.hswebframework.web.api.crud.entity.RecordCreationEntity;
import org.hswebframework.web.api.crud.entity.RecordModifierEntity;
import org.hswebframework.web.crud.annotation.EnableEntityEvent;
import org.hswebframework.web.crud.generator.Generators;
import org.hswebframework.web.validator.CreateGroup;

import javax.persistence.Column;
import javax.persistence.GeneratedValue;
import javax.persistence.Table;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Pattern;
import java.sql.JDBCType;

/**
 * {实体描述}
 *
 * @author {作者}
 * @since {版本}
 */
@Getter
@Setter
@Table(name = "{表名}")
@Comment("{表注释}")
@EnableEntityEvent  // 启用实体事件，支持实体变更监听
public class {实体名}Entity extends GenericEntity<String>
    implements RecordCreationEntity, RecordModifierEntity {

    // 如果需要自定义ID规则，覆盖getId方法
    @Override
    @GeneratedValue(generator = Generators.SNOW_FLAKE)  // 使用雪花算法生成ID
    @Pattern(regexp = "^[0-9a-zA-Z_\\-]+$", groups = CreateGroup.class)
    @Schema(description = "ID(只能由数字,字母,下划线和中划线组成)")
    public String getId() {
        return super.getId();
    }

    @Column(length = 64)
    @NotBlank(message = "名称不能为空", groups = CreateGroup.class)
    @Schema(description = "名称")
    private String name;

    @Column(length = 256)
    @Schema(description = "说明")
    private String description;

    // 枚举类型字段
    @Column(length = 32)
    @EnumCodec
    @ColumnType(javaType = String.class)
    @DefaultValue("enabled")
    @Schema(description = "状态")
    private {枚举类型} state;

    // JSON类型字段(存储Map、List等复杂对象)
    @Column
    @JsonCodec
    @ColumnType(jdbcType = JDBCType.LONGVARCHAR, javaType = String.class)
    @Schema(description = "配置信息")
    private Map<String, Object> configuration;

    // 大文本字段
    @Column
    @ColumnType(jdbcType = JDBCType.CLOB, javaType = String.class)
    @Schema(description = "内容")
    private String content;

    // 创建人信息(只读)
    @Column(updatable = false)
    @Schema(
        description = "创建者ID(只读)",
        accessMode = Schema.AccessMode.READ_ONLY
    )
    private String creatorId;

    @Column(updatable = false)
    @DefaultValue(generator = Generators.CURRENT_TIME)
    @Schema(
        description = "创建时间(只读)",
        accessMode = Schema.AccessMode.READ_ONLY
    )
    private Long createTime;

    // 修改人信息(只读)
    @Column(length = 64)
    @Schema(description = "修改人ID", accessMode = Schema.AccessMode.READ_ONLY)
    private String modifierId;

    @Column
    @DefaultValue(generator = Generators.CURRENT_TIME)
    @Schema(description = "修改时间", accessMode = Schema.AccessMode.READ_ONLY)
    private Long modifyTime;
}
```

### 1.2 树形实体类模板

如果实体需要支持树形结构，继承 `GenericTreeSortSupportEntity`:

```java
public class {实体名}Entity extends GenericTreeSortSupportEntity<String>
    implements RecordCreationEntity {

    @Schema(description = "子节点")
    private List<{实体名}Entity> children;

    // 其他字段...
}
```

### 1.3 枚举类型定义

枚举类需要实现 `I18nEnumDict` 接口以支持国际化和字典功能：

```java
package org.jetlinks.pro.{模块名}.enums;

import lombok.AllArgsConstructor;
import lombok.Getter;
import org.hswebframework.web.dict.I18nEnumDict;

/**
 * {枚举描述}
 * 通过接口 GET /dictionary/{枚举类名}/items 可获取选项内容
 */
@AllArgsConstructor
@Getter
public enum {枚举名} implements I18nEnumDict<String> {

    option1("选项1"),
    option2("选项2"),
    option3("选项3");

    private final String text;

    @Override
    public String getValue() {
        return name();  // 返回枚举名称作为值
    }
}
```

在实体类中使用枚举：

```java
// 单选枚举
@Column(length = 32)
@EnumCodec
@ColumnType(javaType = String.class)
@Schema(description = "状态")
private StateEnum state;

// 多选枚举(使用掩码存储)
@Column
@EnumCodec(toMask = true)  // 使用位掩码存储多个枚举值
@ColumnType(javaType = Long.class, jdbcType = JDBCType.BIGINT)
@Schema(description = "标签")
private StateEnum[] tags;
```

### 1.4 实体注解说明

- `@Table(name = "表名")`: 指定数据库表名
- `@Comment("注释")`: 表或字段的注释
- `@EnableEntityEvent`: 启用实体事件支持
- `@Column`: 标记字段为数据库列
  - `length`: 字段长度
  - `updatable`: 是否可更新(默认true)
  - `nullable`: 是否允许为空(默认true)
- `@ColumnType`: 指定JDBC类型和Java类型
- `@JsonCodec`: 自动将Java对象序列化为JSON存储
- `@EnumCodec`: 枚举类型编解码
  - `toMask = true`: 使用位掩码存储多个枚举值(用于数组)
- `@DefaultValue`: 设置默认值
  - `generator`: 使用生成器(如 `Generators.CURRENT_TIME`)
- `@GeneratedValue`: ID生成策略
  - `generator = Generators.SNOW_FLAKE`: 雪花算法
  - `generator = Generators.UUID`: UUID
- `@Schema`: Swagger文档注解
  - `description`: 字段描述
  - `accessMode`: 访问模式(READ_ONLY/READ_WRITE等)

---

## 二、服务层(Service)开发规范

### 2.1 基础服务类模板

```java
package org.jetlinks.pro.{模块名}.service;

import lombok.extern.slf4j.Slf4j;
import org.hswebframework.web.crud.service.GenericReactiveCrudService;
import org.jetlinks.pro.{模块名}.entity.{实体名}Entity;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

/**
 * {服务描述}
 *
 * @author {作者}
 * @since {版本}
 */
@Service
@Slf4j
public class {实体名}Service extends GenericReactiveCrudService<{实体名}Entity, String> {

    // GenericReactiveCrudService 已提供以下方法:
    // - save(Publisher<E>): 批量保存
    // - insert(Publisher<E>): 批量新增
    // - updateById(K, Mono<E>): 根据ID更新
    // - deleteById(Publisher<K>): 根据ID删除
    // - findById(K): 根据ID查询
    // - createQuery(): 创建查询对象
    // - createUpdate(): 创建更新对象
    // - createDelete(): 创建删除对象

    // 自定义业务方法示例
    public Mono<{实体名}Entity> findByName(String name) {
        return createQuery()
            .where({实体名}Entity::getName, name)
            .fetchOne();
    }

    public Flux<{实体名}Entity> findByState(String state) {
        return createQuery()
            .where({实体名}Entity::getState, state)
            .fetch();
    }

    // 复杂查询示例
    public Mono<Long> countByCondition(String keyword) {
        return createQuery()
            .where()
            .like$({实体名}Entity::getName, keyword)
            .or()
            .like$({实体名}Entity::getDescription, keyword)
            .count();
    }

    // 事务处理示例 (使用 @Transactional 注解)
    @Transactional
    public Mono<Void> batchUpdateState(List<String> ids, String state) {
        return createUpdate()
            .set({实体名}Entity::setState, state)
            .where()
            .in({实体名}Entity::getId, ids)
            .execute()
            .then();
    }
}
```

### 2.2 树形服务类模板

```java
@Service
@Slf4j
public class {实体名}Service extends GenericReactiveTreeSupportCrudService<{实体名}Entity, String> {

    @Override
    public void setChildren({实体名}Entity entity, List<{实体名}Entity> children) {
        entity.setChildren(children);
    }

    @Override
    public IDGenerator<String> getIDGenerator() {
        return IDGenerator.MD5; // 或 IDGenerator.SNOW_FLAKE
    }
}
```

### 2.3 服务层最佳实践

1. **使用Reactive编程**: 所有方法返回 `Mono` 或 `Flux`
2. **链式查询**: 使用 `createQuery()` 构建查询条件
3. **日志记录**: 使用 `@Slf4j` 注解，在关键位置记录日志
4. **异常处理**: 使用 `onErrorResume`、`onErrorReturn` 处理异常
5. **事务控制**: 使用 `@Transactional` 注解管理事务

---

## 二.A、非响应式(Spring MVC)服务层开发规范

### 2.A.1 基础服务类模板(阻塞式)

```java
package org.jetlinks.pro.{模块名}.service;

import lombok.extern.slf4j.Slf4j;
import org.hswebframework.web.crud.service.GenericCrudService;
import org.jetlinks.pro.{模块名}.entity.{实体名}Entity;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

/**
 * {服务描述} - 阻塞式实现
 *
 * @author {作者}
 * @since {版本}
 */
@Service
@Slf4j
public class {实体名}Service extends GenericCrudService<{实体名}Entity, String> {

    // GenericCrudService 已提供以下方法(阻塞式):
    // - save(Collection<E>): 批量保存
    // - insert(Collection<E>): 批量新增
    // - updateById(K, E): 根据ID更新
    // - deleteById(Collection<K>): 根据ID删除
    // - findById(K): 根据ID查询，返回Optional
    // - createQuery(): 创建查询对象
    // - createUpdate(): 创建更新对象
    // - createDelete(): 创建删除对象

    // 自定义业务方法示例
    public Optional<{实体名}Entity> findByName(String name) {
        return createQuery()
            .where({实体名}Entity::getName, name)
            .fetchOne();
    }

    public List<{实体名}Entity> findByState(String state) {
        return createQuery()
            .where({实体名}Entity::getState, state)
            .fetch();
    }

    // 复杂查询示例
    public long countByCondition(String keyword) {
        return createQuery()
            .where()
            .like$({实体名}Entity::getName, keyword)
            .or()
            .like$({实体名}Entity::getDescription, keyword)
            .count();
    }

    // 事务处理示例
    @Transactional
    public void batchUpdateState(List<String> ids, String state) {
        createUpdate()
            .set({实体名}Entity::setState, state)
            .where()
            .in({实体名}Entity::getId, ids)
            .execute();
    }
}
```

### 2.A.2 响应式与阻塞式混合使用

在 Spring MVC 环境中，仍然可以使用响应式 Service，但需要适当转换：

```java
@Service
@Slf4j
public class MixedService {

    @Autowired
    private ReactiveExampleService reactiveService;  // 响应式Service

    @Autowired
    private ExampleService blockingService;          // 阻塞式Service

    // 方式1: 在阻塞式方法中调用响应式Service
    public ExampleEntity getById(String id) {
        return reactiveService
            .findById(id)
            .block();  // 阻塞等待结果
    }

    // 方式2: 使用Reactors工具类转换
    public List<ExampleEntity> queryList() {
        return Reactors.flatMapIterable(
            reactiveService.createQuery().fetch()
        ).block();
    }

    // 方式3: 在响应式方法中调用阻塞式Service
    public Mono<ExampleEntity> getByIdReactive(String id) {
        return Mono.fromCallable(() -> blockingService.findById(id))
            .flatMap(opt -> opt.map(Mono::just).orElse(Mono.empty()));
    }
}
```

### 2.A.3 阻塞式查询示例

```java
// 单条件查询
Optional<Entity> result = service.createQuery()
    .where(Entity::getName, "value")
    .fetchOne();

// 多条件查询(AND) - 返回List
List<Entity> list = service.createQuery()
    .where(Entity::getName, "value1")
    .and()
    .where(Entity::getState, "value2")
    .fetch();

// IN查询
List<Entity> list = service.createQuery()
    .where()
    .in(Entity::getId, Arrays.asList("id1", "id2"))
    .fetch();

// 分页查询
PagerResult<Entity> page = service.createQuery()
    .setParam(queryParam)
    .fetchPaged();

// 统计数量
long count = service.createQuery()
    .where(Entity::getState, "enabled")
    .count();
```

### 2.A.4 阻塞式最佳实践

1. **返回类型**: 使用 `List`, `Optional`, 基本类型等
2. **异常处理**: 使用 try-catch 处理异常
3. **事务管理**: 使用 `@Transactional` 注解
4. **性能考虑**: 阻塞式代码会占用线程，注意线程池配置
5. **混合使用**: 可以在需要时调用响应式Service，使用 `.block()` 转换

---

## 三、控制器层(Controller)开发规范

### 3.1 基础控制器模板

```java
package org.jetlinks.pro.{模块名}.web;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.extern.slf4j.Slf4j;
import org.hswebframework.web.api.crud.entity.PagerResult;
import org.hswebframework.web.api.crud.entity.QueryParamEntity;
import org.hswebframework.web.authorization.annotation.Authorize;
import org.hswebframework.web.authorization.annotation.QueryAction;
import org.hswebframework.web.authorization.annotation.Resource;
import org.hswebframework.web.authorization.annotation.SaveAction;
import org.hswebframework.web.crud.service.ReactiveCrudService;
import org.jetlinks.pro.assets.annotation.AssetsController;
import org.jetlinks.pro.assets.crud.AssetsHolderCrudController;
import org.jetlinks.pro.{模块名}.entity.{实体名}Entity;
import org.jetlinks.pro.{模块名}.service.{实体名}Service;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

/**
 * {控制器描述}
 *
 * @author {作者}
 * @since {版本}
 */
@RestController
@RequestMapping("/{模块路径}/{资源路径}")
@Authorize  // 启用权限控制
@Resource(id = "{资源ID}", name = "{资源名称}", group = "{分组名}")
@Tag(name = "{API标签名}")
@Slf4j
@AllArgsConstructor
@Getter
@AssetsController(type = "{资产类型}")  // 启用资产权限控制
public class {实体名}Controller implements AssetsHolderCrudController<{实体名}Entity, String> {

    private final {实体名}Service service;

    // AssetsHolderCrudController 已提供以下RESTful接口:
    // GET    /{id}           - 根据ID查询 (getById)
    // GET    /_query         - 查询列表 (query)
    // POST   /_query         - 分页查询 (queryPager)
    // GET    /_count         - 统计数量 (count)
    // POST   /               - 新增/批量保存 (save)
    // PUT    /{id}           - 更新 (update)
    // PATCH  /_batch         - 批量修改 (saveBatch)
    // DELETE /{id}           - 删除 (delete)

    // 自定义接口示例

    @GetMapping("/{id}/detail")
    @QueryAction  // 查询操作，需要query权限
    @Operation(summary = "获取详情信息")
    public Mono<{实体名}Entity> getDetail(
            @PathVariable @Parameter(description = "ID") String id) {
        return service.findById(id)
            .switchIfEmpty(Mono.error(new NotFoundException("数据不存在")));
    }

    @PostMapping("/{id}/_enable")
    @SaveAction  // 保存操作，需要save权限
    @Operation(summary = "启用")
    public Mono<Void> enable(@PathVariable String id) {
        return service
            .createUpdate()
            .set({实体名}Entity::getState, "enabled")
            .where({实体名}Entity::getId, id)
            .execute()
            .then();
    }

    @PostMapping("/{id}/_disable")
    @SaveAction
    @Operation(summary = "禁用")
    public Mono<Void> disable(@PathVariable String id) {
        return service
            .createUpdate()
            .set({实体名}Entity::getState, "disabled")
            .where({实体名}Entity::getId, id)
            .execute()
            .then();
    }

    @PostMapping("/_query/no-paging")
    @QueryAction
    @Operation(summary = "不分页查询")
    public Flux<{实体名}Entity> queryNoPaging(@RequestBody Mono<QueryParamEntity> query) {
        return query.flatMapMany(service::query);
    }

    @GetMapping("/{id}/_validate")
    @QueryAction
    @AssetsController(ignore = true)  // 忽略资产权限控制
    @Operation(summary = "验证ID是否存在")
    public Mono<Boolean> validateId(@PathVariable String id) {
        return service
            .findById(id)
            .hasElement();
    }
}
```

### 3.2 树形结构控制器

```java
@GetMapping("/_tree")
@QueryAction
@Operation(summary = "查询树形结构")
@Authorize(merge = false)  // 不合并权限检查
public Flux<{实体名}Entity> queryTree(@Parameter(hidden = true) QueryParamEntity query) {
    return service
        .createQuery()
        .setParam(query)
        .fetch()
        .collectList()
        .flatMapMany(all -> Flux.fromIterable(
            TreeSupportEntity.list2tree(all, {实体名}Entity::setChildren)
        ));
}
```

### 3.3 关联资产权限控制

```java
// 当实体与其他资产关联时，使用 CorrelatesAssetsHolderCrudController
public class {实体名}Controller implements CorrelatesAssetsHolderCrudController<{实体名}Entity, String> {

    @NotNull
    @Override
    public AssetType getAssetType() {
        return {关联的资产类型};  // 如 DeviceAssetType.product
    }

    @NotNull
    @Override
    public Function<{实体名}Entity, ?> getAssetIdMapper() {
        return {实体名}Entity::get关联资产ID;
    }

    @NotNull
    @Override
    public String getAssetProperty() {
        return "关联资产ID字段名";
    }
}
```

### 3.4 多资产权限控制

```java
@PostMapping("/{id}/items/_bind")
@SaveAction
@Operation(summary = "绑定项目")
@MultiAssetsController({
    @AssetsController(type = "资产类型1"),
    @AssetsController(type = "资产类型2", assetIdIndex = 1)
})
public Mono<Void> bindItems(
        @PathVariable String id,
        @RequestBody Mono<List<String>> itemIds) {
    // 实现绑定逻辑
    return itemIds.flatMap(ids -> service.bind(id, ids));
}
```

---

## 三.A、非响应式(Spring MVC)控制器层开发规范

### 3.A.1 基础控制器模板(阻塞式)

```java
package org.jetlinks.pro.{模块名}.web;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.extern.slf4j.Slf4j;
import org.hswebframework.web.api.crud.entity.PagerResult;
import org.hswebframework.web.api.crud.entity.QueryParamEntity;
import org.hswebframework.web.authorization.annotation.Authorize;
import org.hswebframework.web.authorization.annotation.QueryAction;
import org.hswebframework.web.authorization.annotation.Resource;
import org.hswebframework.web.authorization.annotation.SaveAction;
import org.hswebframework.web.crud.service.CrudService;
import org.jetlinks.pro.assets.annotation.AssetsController;
import org.jetlinks.pro.assets.crud.BlockingAssetsHolderCrudController;
import org.jetlinks.pro.{模块名}.entity.{实体名}Entity;
import org.jetlinks.pro.{模块名}.service.{实体名}Service;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * {控制器描述} - 阻塞式实现
 *
 * @author {作者}
 * @since {版本}
 */
@RestController
@RequestMapping("/{模块路径}/{资源路径}")
@Authorize  // 启用权限控制
@Resource(id = "{资源ID}", name = "{资源名称}", group = "{分组名}")
@Tag(name = "{API标签名}")
@Slf4j
@AllArgsConstructor
@Getter
@AssetsController(type = "{资产类型}")  // 启用资产权限控制
public class {实体名}Controller implements BlockingAssetsHolderCrudController<{实体名}Entity, String> {

    private final {实体名}Service service;

    // BlockingAssetsHolderCrudController 已提供以下RESTful接口:
    // GET    /{id}           - 根据ID查询 (getById)
    // GET    /_query         - 查询列表 (query)
    // POST   /_query         - 分页查询 (queryPager)
    // GET    /_count         - 统计数量 (count)
    // POST   /               - 新增/批量保存 (save)
    // PUT    /{id}           - 更新 (update)
    // PATCH  /_batch         - 批量修改 (saveBatch)
    // DELETE /{id}           - 删除 (delete)

    // 自定义接口示例

    @GetMapping("/{id}/detail")
    @QueryAction  // 查询操作，需要query权限
    @Operation(summary = "获取详情信息")
    public {实体名}Entity getDetail(
            @PathVariable @Parameter(description = "ID") String id) {
        return service.findById(id)
            .orElseThrow(() -> new NotFoundException("数据不存在"));
    }

    @PostMapping("/{id}/_enable")
    @SaveAction  // 保存操作，需要save权限
    @Operation(summary = "启用")
    public void enable(@PathVariable String id) {
        service
            .createUpdate()
            .set({实体名}Entity::getState, "enabled")
            .where({实体名}Entity::getId, id)
            .execute();
    }

    @PostMapping("/{id}/_disable")
    @SaveAction
    @Operation(summary = "禁用")
    public void disable(@PathVariable String id) {
        service
            .createUpdate()
            .set({实体名}Entity::getState, "disabled")
            .where({实体名}Entity::getId, id)
            .execute();
    }

    @PostMapping("/_query/no-paging")
    @QueryAction
    @Operation(summary = "不分页查询")
    public List<{实体名}Entity> queryNoPaging(@RequestBody QueryParamEntity query) {
        return service.query(query);
    }

    @GetMapping("/{id}/_validate")
    @QueryAction
    @AssetsController(ignore = true)  // 忽略资产权限控制
    @Operation(summary = "验证ID是否存在")
    public boolean validateId(@PathVariable String id) {
        return service.findById(id).isPresent();
    }
}
```

### 3.A.2 阻塞式与响应式混合使用

在 Spring MVC Controller 中可以混合使用响应式和阻塞式：

```java
@RestController
@RequestMapping("/mixed")
@Resource(id = "mixed", name = "混合示例")
@Tag(name = "混合模式示例")
public class MixedController {

    @Autowired
    private ExampleService blockingService;      // 阻塞式Service

    @Autowired
    private ReactiveExampleService reactiveService;  // 响应式Service

    // 方式1: 阻塞式接口
    @GetMapping("/blocking/{id}")
    public ExampleEntity getBlocking(@PathVariable String id) {
        return blockingService.findById(id)
            .orElseThrow(() -> new NotFoundException());
    }

    // 方式2: 响应式接口 - 返回Mono
    @GetMapping("/reactive/{id}")
    public Mono<ExampleEntity> getReactive(@PathVariable String id) {
        return reactiveService.findById(id);
    }

    // 方式3: 响应式流式接口 - 返回Flux
    @GetMapping("/stream")
    public Flux<ExampleEntity> stream() {
        return Flux
            .interval(Duration.ofSeconds(1))
            .flatMap(i -> reactiveService.createQuery().fetch())
            .take(10);
    }

    // 方式4: 在阻塞式接口中调用响应式Service
    @GetMapping("/blocking-with-reactive/{id}")
    public ExampleEntity getBlockingWithReactive(@PathVariable String id) {
        return reactiveService
            .findById(id)
            .block();  // 阻塞等待结果
    }
}
```

### 3.A.3 关联资产权限控制(阻塞式)

```java
// 当实体与其他资产关联时，使用 BlockingCorrelatesAssetsHolderCrudController
public class {实体名}Controller implements BlockingCorrelatesAssetsHolderCrudController<{实体名}Entity, String> {

    @NotNull
    @Override
    public AssetType getAssetType() {
        return {关联的资产类型};
    }

    @NotNull
    @Override
    public Function<{实体名}Entity, ?> getAssetIdMapper() {
        return {实体名}Entity::get关联资产ID;
    }

    @NotNull
    @Override
    public String getAssetProperty() {
        return "关联资产ID字段名";
    }
}
```

### 3.A.4 阻塞式最佳实践

1. **返回类型**: 使用普通Java对象、List、Optional等
2. **异常处理**: 使用标准的 try-catch 或抛出异常
3. **性能考虑**: 阻塞式会占用线程，适合低并发场景
4. **流式响应**: 可以返回 Flux/Mono 实现流式响应，前端通过 `application/x-ndjson` 接收
5. **虚拟线程**: Java 21+ 可开启虚拟线程提升阻塞式性能

---

### 3.5 控制器注解说明

#### 权限控制注解
- `@Authorize`: 启用权限控制(类或方法级别)
  - `merge = false`: 不合并权限检查
- `@Resource`: 定义资源
  - `id`: 资源ID(必填)
  - `name`: 资源名称(必填)
  - `group`: 资源分组
- `@QueryAction`: 查询操作，需要 `query` 权限
- `@SaveAction`: 保存操作，需要 `save` 权限
- `@DeleteAction`: 删除操作，需要 `delete` 权限

#### 资产权限注解
- `@AssetsController`: 资产权限控制
  - `type`: 资产类型
  - `assetIdIndex`: 资产ID在方法参数中的索引(默认0)
  - `assetObjectIndex`: 资产对象在方法参数中的索引(默认-1)
  - `property`: 资产ID属性名(默认"id")
  - `required`: 是否必须有资产权限(默认false)
  - `autoBind`: 保存时自动绑定资产(默认false)
  - `autoUnbind`: 删除时自动解绑资产(默认false)
  - `ignore`: 忽略资产权限控制(默认false)
  - `ignoreQuery`: 忽略查询时的资产权限控制(默认false)
  - `validate`: 是否验证资产权限(默认true)
  - `allowAssetNotExist`: 允许资产不存在(默认false)
  - `permission`: 自定义权限标识

- `@MultiAssetsController`: 多资产权限控制

#### Swagger文档注解
- `@Tag`: API分组标签
- `@Operation`: 接口描述
  - `summary`: 简要描述
  - `description`: 详细描述
- `@Parameter`: 参数描述
  - `description`: 参数说明
  - `hidden`: 是否隐藏(默认false)

---

## 四、权限控制最佳实践

### 4.1 权限层级

```
资源级别 (@Resource)
    ↓
操作级别 (@QueryAction, @SaveAction, @DeleteAction)
    ↓
资产级别 (@AssetsController, @DeviceAsset, @ProductAsset)
```

### 4.2 常见权限配置

#### 标准CRUD权限
```java
@RestController
@RequestMapping("/api/resource")
@Authorize
@Resource(id = "resource-id", name = "资源名称")
@AssetsController(type = "资产类型")
public class ResourceController implements AssetsHolderCrudController<Entity, String> {
    // 继承接口后自动获得增删改查的权限控制
}
```

#### 公开接口(无需权限)
```java
@GetMapping("/public/info")
@Authorize(ignore = true)  // 忽略权限检查
public Mono<Info> getPublicInfo() {
    return ...;
}
```

#### 忽略资产权限
```java
@GetMapping("/validate")
@QueryAction
@AssetsController(ignore = true)  // 忽略资产权限，但仍需基础query权限
public Mono<Boolean> validate() {
    return ...;
}
```

#### 仅需要登录
```java
@GetMapping("/me/info")
@Authorize(merge = false)  // 只验证登录状态，不验证具体权限
public Mono<Info> getMyInfo() {
    return ...;
}
```

### 4.3 自定义资产类型

```java
package org.jetlinks.pro.{模块名}.assets;

import org.jetlinks.pro.assets.annotation.AssetsController;
import org.springframework.core.annotation.AliasFor;
import java.lang.annotation.*;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@AssetsController(type = "{自定义资产类型}")
public @interface {自定义}Asset {

    @AliasFor(annotation = AssetsController.class)
    int assetIdIndex() default 0;

    @AliasFor(annotation = AssetsController.class)
    boolean required() default false;

    @AliasFor(annotation = AssetsController.class)
    boolean autoBind() default false;

    @AliasFor(annotation = AssetsController.class)
    boolean ignore() default false;

    // 其他配置项...
}
```

---

## 五、常用查询模式

### 5.1 基础查询

```java
// 单条件查询
service.createQuery()
    .where(Entity::getName, "value")
    .fetchOne();

// 多条件查询(AND)
service.createQuery()
    .where(Entity::getName, "value1")
    .and()
    .where(Entity::getState, "value2")
    .fetch();

// 多条件查询(OR)
service.createQuery()
    .where(Entity::getName, "value1")
    .or()
    .where(Entity::getName, "value2")
    .fetch();

// IN查询
service.createQuery()
    .where()
    .in(Entity::getId, Arrays.asList("id1", "id2"))
    .fetch();

// 模糊查询
service.createQuery()
    .where()
    .like$(Entity::getName, "keyword")  // %keyword%
    .fetch();

// 范围查询
service.createQuery()
    .where()
    .gte(Entity::getCreateTime, startTime)  // >=
    .lte(Entity::getCreateTime, endTime)    // <=
    .fetch();
```

### 5.2 分页查询

```java
// 使用 QueryParamEntity
public Mono<PagerResult<Entity>> queryPager(QueryParamEntity query) {
    return service
        .createQuery()
        .setParam(query)
        .fetchPaged();
}
```

### 5.3 关联查询

```java
// 左关联
service.createQuery()
    .leftJoin(OtherEntity.class, (query, otherQuery) -> {
        return query.where(Entity::getOtherId, otherQuery.getExpr(OtherEntity::getId));
    })
    .fetch();
```

### 5.4 分组统计

```java
service.createQuery()
    .groupBy(Entity::getType)
    .select(Entity::getType)
    .count("total")
    .fetch();
```

---

## 六、响应式编程规范

### 6.1 Mono和Flux的选择

- `Mono<T>`: 返回0或1个元素
  - 单个对象查询
  - 保存/更新/删除操作
  - 统计操作

- `Flux<T>`: 返回0到N个元素
  - 列表查询
  - 批量操作
  - 流式处理

### 6.2 常用操作符

```java
// flatMap: 转换并扁平化
mono.flatMap(entity -> service.doSomething(entity))

// map: 简单转换
mono.map(entity -> entity.getName())

// flatMapMany: Mono转Flux
mono.flatMapMany(entity -> Flux.fromIterable(entity.getItems()))

// switchIfEmpty: 空值处理
mono.switchIfEmpty(Mono.error(new NotFoundException()))

// defaultIfEmpty: 提供默认值
mono.defaultIfEmpty(defaultEntity)

// zipWith: 组合两个Mono
mono1.zipWith(mono2, (a, b) -> new Result(a, b))

// then: 忽略结果，返回Mono<Void>
mono.then()

// thenReturn: 返回固定值
mono.thenReturn(true)

// doOnNext: 副作用操作(不修改流)
flux.doOnNext(entity -> log.info("处理: {}", entity))

// collectList: Flux转List
flux.collectList()

// onErrorResume: 异常处理
mono.onErrorResume(e -> {
    log.error("发生错误", e);
    return Mono.empty();
})

// onErrorReturn: 异常时返回默认值
mono.onErrorReturn(defaultValue)
```

### 6.3 响应式最佳实践

1. **避免阻塞操作**: 不要在Reactor链中使用 `.block()`
2. **异常处理**: 使用 `onErrorResume`、`onErrorReturn` 而不是 try-catch
3. **日志记录**: 使用 `doOnNext`、`doOnError` 而不是直接log
4. **链式调用**: 保持链式调用的可读性，适当换行
5. **延迟执行**: 使用 `Mono.defer()` 或 `Flux.defer()` 延迟执行

---

## 七、国际化支持

### 7.1 消息文件

在 `src/main/resources/i18n/{模块名}/` 目录下创建:
- `messages_zh.properties`: 中文
- `messages_en.properties`: 英文

```properties
# messages_zh.properties
error.data_not_found=数据不存在
error.name_duplicate=名称已存在
success.save=保存成功
```

### 7.2 使用国际化消息

```java
import org.hswebframework.web.i18n.LocaleUtils;

// 在Service或Controller中
LocaleUtils.resolveMessageReactive("error.data_not_found")
    .flatMap(msg -> Mono.error(new BusinessException(msg)));

// 带参数的消息
LocaleUtils.resolveMessageReactive("error.name_duplicate", "名称A")
    .flatMap(msg -> Mono.error(new BusinessException(msg)));
```

### 7.3 实体国际化

```java
// 实现 MultipleI18nSupportEntity 接口
public class Entity extends GenericEntity<String> implements MultipleI18nSupportEntity {

    @Schema(title = "国际化信息定义")
    @Column
    @JsonCodec
    @ColumnType(jdbcType = JDBCType.LONGVARCHAR, javaType = String.class)
    private Map<String, Map<String, String>> i18nMessages;

    public String getI18nName() {
        return getI18nMessage("name", name);
    }
}
```

---

## 八、异常处理规范

### 8.1 常用异常类型

```java
import org.hswebframework.web.exception.BusinessException;
import org.hswebframework.web.exception.NotFoundException;
import org.hswebframework.web.exception.ValidationException;
import org.jetlinks.pro.exception.AccessDenyException;

// 业务异常
throw new BusinessException("错误消息");
throw new BusinessException("error.i18n_key");

// 资源不存在
throw new NotFoundException("数据不存在");

// 验证异常
throw new ValidationException("error.validation_failed");

// 权限拒绝
throw new AccessDenyException("资源名", Collections.emptySet());
```

### 8.2 响应式异常处理

```java
return service.findById(id)
    .switchIfEmpty(Mono.error(new NotFoundException("数据不存在")))
    .flatMap(entity -> {
        if (!entity.isValid()) {
            return Mono.error(new ValidationException("error.invalid_data"));
        }
        return service.save(entity);
    })
    .onErrorResume(BusinessException.class, e -> {
        log.error("业务异常", e);
        return Mono.error(e);  // 继续抛出
    })
    .onErrorResume(Exception.class, e -> {
        log.error("系统异常", e);
        return Mono.error(new BusinessException("系统错误", e));
    });
```

---

## 九、测试规范

### 9.1 测试类模板

```java
package org.jetlinks.pro.{模块名}.service;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import reactor.test.StepVerifier;

@SpringBootTest
class {实体名}ServiceTest {

    @Autowired
    private {实体名}Service service;

    @Test
    void testSave() {
        {实体名}Entity entity = new {实体名}Entity();
        entity.setName("测试");

        StepVerifier.create(service.save(entity))
            .expectNextMatches(saved -> saved.getId() != null)
            .verifyComplete();
    }

    @Test
    void testFindById() {
        StepVerifier.create(service.findById("test-id"))
            .expectNextMatches(entity -> "test-id".equals(entity.getId()))
            .verifyComplete();
    }
}
```

### 9.2 Controller测试

```java
@SpringBootTest
@AutoConfigureWebTestClient
class {实体名}ControllerTest {

    @Autowired
    private WebTestClient client;

    @Test
    void testQuery() {
        client.get()
            .uri("/{模块路径}/{资源路径}/_query")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList({实体名}Entity.class)
            .hasSize(10);
    }
}
```

---

## 十、代码生成指南(供AI使用)

### 10.1 快速生成CRUD模块步骤

当用户要求创建一个新的CRUD模块时，按照以下步骤生成代码:

1. **明确需求**
   - 模块名称(如: notify-manager, device-manager)
   - 实体名称(如: NotifyTemplate, DeviceCategory)
   - 表名
   - 字段列表(名称、类型、是否必填、描述)
   - 是否需要树形结构
   - 资产类型(如果需要资产权限控制)
   - **编程模式**: 响应式(WebFlux) 或 阻塞式(Spring MVC)

2. **创建目录结构**
```
modules/{模块名}/src/main/java/org/jetlinks/pro/{模块名}/
  ├── entity/        # 实体类
  ├── service/       # 服务类
  ├── web/           # 控制器
  ├── enums/         # 枚举类
  └── assets/        # 资产类型定义(如需要)
```

3. **生成实体类**
   - 根据字段列表生成完整的Entity类
   - 添加必要的注解(@Table, @Column, @Schema等)
   - 实现RecordCreationEntity和RecordModifierEntity接口
   - 如果是树形结构，继承GenericTreeSortSupportEntity

4. **生成服务类**
   - **响应式**: 继承GenericReactiveCrudService或GenericReactiveTreeSupportCrudService
   - **阻塞式**: 继承GenericCrudService
   - 添加@Service和@Slf4j注解
   - 根据业务需求添加自定义方法

5. **生成控制器类**
   - **响应式**: 实现AssetsHolderCrudController接口
   - **阻塞式**: 实现BlockingAssetsHolderCrudController接口
   - 添加完整的权限注解(@Resource, @Authorize, @AssetsController)
   - 添加Swagger注解(@Tag, @Operation)
   - 根据需求添加自定义接口

6. **生成国际化文件**
   - 在resources/i18n/{模块名}/目录下创建消息文件
   - 添加常用的错误消息和提示信息

7. **生成测试类**
   - 为Service和Controller生成基础测试类

### 10.2 代码生成检查清单

- [ ] Entity类包含所有必需注解
- [ ] Entity类实现了RecordCreationEntity和RecordModifierEntity
- [ ] Service类继承了正确的基类
  - [ ] 响应式: GenericReactiveCrudService
  - [ ] 阻塞式: GenericCrudService
- [ ] Controller实现了正确的接口
  - [ ] 响应式: AssetsHolderCrudController
  - [ ] 阻塞式: BlockingAssetsHolderCrudController
- [ ] 权限注解配置完整(@Resource, @Authorize, @AssetsController)
- [ ] Swagger文档注解完整(@Tag, @Operation, @Parameter)
- [ ] 返回类型正确
  - [ ] 响应式: 所有方法返回Mono或Flux
  - [ ] 阻塞式: 返回普通Java对象、List、Optional等
- [ ] 异常处理完整
  - [ ] 响应式: 使用onErrorResume/onErrorReturn
  - [ ] 阻塞式: 使用try-catch或抛出异常
- [ ] 包含基础的测试用例

### 10.3 常见问题处理

**Q: 如何处理枚举类型?**
A:
```java
// 1. 创建枚举类实现I18nEnumDict接口
public enum StateEnum implements I18nEnumDict<String> {
    enabled("启用"),
    disabled("禁用");

    private final String text;
    StateEnum(String text) { this.text = text; }
    public String getValue() { return name(); }
}

// 2. 在Entity中使用
@Column(length = 32)
@EnumCodec
@ColumnType(javaType = String.class)
private StateEnum state;

// 3. 多选枚举使用掩码
@Column
@EnumCodec(toMask = true)
@ColumnType(javaType = Long.class, jdbcType = JDBCType.BIGINT)
private StateEnum[] tags;
```

**Q: 如何存储JSON对象?**
A: 使用Map<String, Object>类型，添加@JsonCodec和@ColumnType注解
```java
@Column
@JsonCodec
@ColumnType(jdbcType = JDBCType.LONGVARCHAR, javaType = String.class)
private Map<String, Object> configuration;
```

**Q: 如何实现关联查询?**
A: 使用leftJoin或innerJoin方法，或在Service中分别查询后组装
```java
// 响应式
return service.createQuery()
    .leftJoin(OtherEntity.class, (main, other) ->
        main.where(Entity::getOtherId, other.getExpr(OtherEntity::getId)))
    .fetch();

// 阻塞式
List<Entity> result = service.createQuery()
    .leftJoin(OtherEntity.class, (main, other) ->
        main.where(Entity::getOtherId, other.getExpr(OtherEntity::getId)))
    .fetch();
```

**Q: 如何处理大文本字段?**
A: 使用@ColumnType(jdbcType = JDBCType.CLOB)
```java
@Column
@ColumnType(jdbcType = JDBCType.CLOB, javaType = String.class)
private String content;
```

**Q: 如何实现级联删除?**
A: 在Service中重写delete方法，添加级联删除逻辑
```java
// 响应式
@Override
public Mono<Integer> deleteById(Publisher<String> idPublisher) {
    return Flux.from(idPublisher)
        .flatMap(id ->
            // 先删除关联数据
            relatedService.deleteByParentId(id)
                // 再删除主数据
                .then(super.deleteById(Mono.just(id)))
        )
        .reduce(0, Integer::sum);
}

// 阻塞式
@Override
public int deleteById(Collection<String> ids) {
    ids.forEach(id -> relatedService.deleteByParentId(id));
    return super.deleteById(ids);
}
```

**Q: 如何在阻塞式代码中调用响应式Service?**
A: 使用 `.block()` 或 `Reactors` 工具类
```java
// 方式1: 直接block
Entity entity = reactiveService.findById(id).block();

// 方式2: 使用Reactors工具类
List<Entity> list = Reactors.flatMapIterable(
    reactiveService.createQuery().fetch()
).block();
```

**Q: 如何在响应式代码中调用阻塞式Service?**
A: 使用 `Mono.fromCallable()` 包装
```java
public Mono<Entity> getByIdReactive(String id) {
    return Mono.fromCallable(() -> blockingService.findById(id))
        .flatMap(opt -> opt.map(Mono::just).orElse(Mono.empty()));
}
```

---

## 十一、性能优化建议

### 11.1 数据库优化
- 为常用查询字段添加索引
- 避免查询过多字段，使用select指定字段
- 批量操作使用bufferSize控制批次大小

### 11.2 响应式优化
- 使用Flux.buffer()控制批处理大小
- 避免在Reactor链中使用.block()
- 使用.cache()缓存可重复使用的Mono/Flux

### 11.3 缓存策略
```java
private Mono<Entity> cachedMono = service.findById(id).cache();

// 或使用Spring Cache
@Cacheable(value = "entity", key = "#id")
public Mono<Entity> findById(String id) {
    return repository.findById(id);
}
```

---

## 十二、安全规范

### 12.1 输入验证
- 使用jakarta.validation注解验证输入
- 在Entity中定义验证规则
- 使用CreateGroup和UpdateGroup区分创建和更新验证

### 12.2 SQL注入防护
- 使用参数化查询，避免字符串拼接
- EasyORM自动提供参数化查询支持

### 12.3 XSS防护
- 前端输出时进行HTML转义
- 后端存储时不做转义，保持原始数据

---

## 附录: 快速参考

### 常用注解速查表

| 注解 | 用途 | 位置 |
|-----|------|-----|
| @Table | 指定表名 | Entity类 |
| @Column | 标记字段 | Entity字段 |
| @Schema | API文档 | Entity字段/Controller方法 |
| @Service | 服务类 | Service类 |
| @RestController | REST控制器 | Controller类 |
| @Resource | 定义资源 | Controller类 |
| @Authorize | 启用权限 | Controller类/方法 |
| @AssetsController | 资产权限 | Controller类/方法 |
| @QueryAction | 查询操作 | Controller方法 |
| @SaveAction | 保存操作 | Controller方法 |

### 常用类速查表

| 类名 | 用途 | 模式 |
|-----|------|-----|
| GenericEntity | 基础实体类 | 通用 |
| GenericTreeSortSupportEntity | 树形实体类 | 通用 |
| RecordCreationEntity | 记录创建人接口 | 通用 |
| RecordModifierEntity | 记录修改人接口 | 通用 |
| I18nEnumDict | 枚举字典接口 | 通用 |
| **服务层** | | |
| GenericReactiveCrudService | 响应式服务基类 | 响应式 |
| GenericReactiveTreeSupportCrudService | 响应式树形服务基类 | 响应式 |
| GenericCrudService | 阻塞式服务基类 | 阻塞式 |
| **控制器层** | | |
| AssetsHolderCrudController | 响应式控制器接口 | 响应式 |
| CorrelatesAssetsHolderCrudController | 响应式关联资产控制器 | 响应式 |
| BlockingAssetsHolderCrudController | 阻塞式控制器接口 | 阻塞式 |
| BlockingCorrelatesAssetsHolderCrudController | 阻塞式关联资产控制器 | 阻塞式 |
| **查询与结果** | | |
| QueryParamEntity | 查询参数 | 通用 |
| PagerResult | 分页结果 | 通用 |
| **响应式类型** | | |
| Mono | 0-1个元素的发布者 | 响应式 |
| Flux | 0-N个元素的发布者 | 响应式 |
| **工具类** | | |
| Reactors | 响应式工具类 | 通用 |

---

## 十三、编程模式选择建议

### 13.1 响应式编程(WebFlux)适用场景
- ✅ 高并发、IO密集型应用
- ✅ 需要背压(backpressure)控制
- ✅ 长连接、流式数据处理
- ✅ 微服务间通信密集
- ✅ 资源利用率要求高

### 13.2 阻塞式编程(Spring MVC)适用场景
- ✅ 传统业务系统、管理后台
- ✅ 并发量不高的场景
- ✅ 团队不熟悉响应式编程
- ✅ 大量使用阻塞式第三方库
- ✅ 需要快速开发，代码简单直观
- ✅ Java 21+ 可使用虚拟线程提升性能

### 13.3 模式对比

| 特性 | 响应式(WebFlux) | 阻塞式(Spring MVC) |
|-----|----------------|-------------------|
| 并发模型 | 非阻塞、事件驱动 | 线程池模型 |
| 线程占用 | 少量线程处理大量请求 | 每个请求占用一个线程 |
| 性能 | 高并发下性能更好 | 低并发下性能足够 |
| 代码复杂度 | 较高，需要理解响应式概念 | 较低，传统编程模式 |
| 学习成本 | 高 | 低 |
| 调试难度 | 较难，异步调用栈 | 简单，同步调用栈 |
| 返回类型 | Mono/Flux | Object/List/Optional |
| 适用场景 | IO密集型、高并发 | CPU密集型、低并发 |

---

## 总结

本规范涵盖了JetLinks项目中CRUD模块开发的完整流程，包括:
1. 实体类的定义和注解使用
2. **响应式和阻塞式**两种服务层实现方式
3. **响应式和阻塞式**两种控制器实现方式
4. 权限控制和资产管理
5. 响应式编程最佳实践
6. 阻塞式编程最佳实践
7. 测试和文档编写
8. 编程模式选择建议

遵循本规范可以快速、规范地开发出高质量的业务代码。

**AI使用建议**:
- 根据用户需求，**优先询问用户选择响应式还是阻塞式模式**
- 如果用户未明确说明，**默认使用响应式模式**（项目主流）
- 按照本规范的对应模板生成代码
- 确保生成的代码包含所有必需的注解和配置
- 生成代码时考虑完整性: Entity + Service + Controller + 测试
- 提供清晰的代码说明和使用示例
- 必要时说明两种模式的差异和选择建议

