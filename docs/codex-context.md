# ThingsBoard 二开 Codex 上下文模板

> 用法：把本文件复制到 ThingsBoard 项目根目录，建议命名为 `docs/codex-context.md` 或 `CODEX_CONTEXT.md`。之后每次让 Codex 处理需求时，先说：`先阅读 docs/codex-context.md，再按模板约束处理本次需求。`

## 1. 项目定位

本项目是基于 ThingsBoard 的二次开发项目，包含：

- PC 管理后台前端
- 后端 Java 服务
- 数据库实体、DAO、Service、Controller、权限、规则链等相关逻辑
- 部署、配置、构建脚本

开发原则：

- 优先保持 ThingsBoard 原有架构、代码风格和命名习惯。
- 优先小范围修改，不做无关重构。
- 不随意升级依赖，不格式化无关文件。
- 修改前先定位相关文件，确认影响范围。
- 遇到不确定的业务规则，先说明假设，再继续实现。

## 2. Token 节省规则

Codex 每次处理任务时必须遵守：

1. 先用 `rg` / 文件名搜索定位，不要全仓库展开阅读。
2. 每轮最多先读取 3-8 个最相关文件。
3. 不读取 `target/`、`node_modules/`、`dist/`、`.idea/`、`.git/`、构建产物、日志文件。
4. 只在需要时读取大型文件的局部片段。
5. 先输出“将要查看/修改的文件列表”，再进行改动。
6. 如果需求跨前后端，先画出调用链：页面 -> service/api -> controller -> service -> dao/entity。
7. 不要为了理解全局而扫描整个 ThingsBoard。

推荐提示词：

```text
请按省 token 模式处理：
1. 先用 rg 定位相关文件；
2. 只读取必要文件；
3. 先说明前后端调用链和准备修改的文件；
4. 保持 ThingsBoard 原有风格；
5. 不做无关重构；
6. 修改后给出验证方式。
需求是：……
```

## 3. 常见目录速查

根据实际项目修改下面路径：

```text
前端 PC:
  ui-ngx/
  ui-ngx/src/app/

后端服务:
  application/
  common/
  dao/
  service/
  transport/
  rule-engine/

数据库迁移:
  dao/src/main/resources/sql/
  application/src/main/data/

配置:
  application/src/main/resources/
  docker/
  k8s/

构建:
  pom.xml
  ui-ngx/package.json
```

## 4. 前端二开规则

前端通常是 Angular + TypeScript，处理需求时优先定位：

```text
路由:
  ui-ngx/src/app/modules/**/**.routes.ts
  ui-ngx/src/app/modules/**/**-routing.module.ts

页面组件:
  ui-ngx/src/app/modules/
  ui-ngx/src/app/shared/

服务/API:
  ui-ngx/src/app/core/http/
  ui-ngx/src/app/core/api/
  ui-ngx/src/app/shared/services/

模型类型:
  ui-ngx/src/app/shared/models/

国际化:
  ui-ngx/src/assets/locale/

样式:
  对应组件 .scss
```

前端修改要求：

- 优先复用现有组件、表格、弹窗、表单、权限指令、翻译 key。
- 新增页面时同时检查路由、菜单、权限、国际化。
- 新增接口调用时优先在现有 HTTP service 中扩展。
- 不直接硬编码后端 URL。
- 不随意绕过 ThingsBoard 的 authority / tenant / customer 权限体系。

前端定位提示词：

```text
这是 ThingsBoard PC 前端任务。请先定位：
1. 页面组件；
2. 路由；
3. 菜单入口；
4. 前端 service；
5. model 类型；
6. i18n 文案。
先不要改代码，只列出相关文件和调用关系。
```

## 5. 后端二开规则

后端通常是 Java + Spring，处理需求时优先定位：

```text
Controller:
  application/src/main/java/**/controller/

Service:
  application/src/main/java/**/service/
  service/src/main/java/

DAO:
  dao/src/main/java/

Entity / Model:
  common/src/main/java/
  dao/src/main/java/

权限:
  SecurityUser
  Authority
  access control
  tenant/customer/user 相关校验

数据库:
  dao/src/main/resources/sql/
```

后端修改要求：

- Controller 只做参数、权限、调用编排，业务逻辑放 Service。
- DAO 层遵循现有 Repository / Dao 模式。
- 新增接口必须考虑 tenantId、customerId、authority 权限。
- 新增表或字段必须补数据库迁移脚本。
- 返回对象优先复用现有 DTO / Model。
- 异常处理、分页、排序、查询条件使用 ThingsBoard 现有模式。

后端定位提示词：

```text
这是 ThingsBoard 后端任务。请先定位：
1. REST Controller；
2. Service 接口和实现；
3. DAO/Repository；
4. Entity/DTO；
5. 权限校验位置；
6. 数据库迁移脚本位置。
先不要改代码，只列出调用链和可能影响范围。
```

## 6. 前后端联调任务模板

适合新增页面、新增字段、新增接口、列表查询、详情页扩展：

```text
需求：
【写清楚业务目标】

数据来源：
【来自已有接口 / 新接口 / 数据库新字段 / 第三方接口】

前端入口：
【菜单 / 页面 / 弹窗 / tab / 表格按钮】

后端接口：
【GET/POST/PUT/DELETE，大概路径】

权限规则：
【租户管理员 / 客户 / 系统管理员 / 普通用户】

数据库变化：
【无 / 新增字段 / 新增表 / 迁移旧数据】

请按以下步骤处理：
1. 先用 rg 定位相关文件；
2. 给出前后端调用链；
3. 说明准备修改哪些文件；
4. 经确认后再实现；
5. 实现后给出验证步骤。
```

## 7. 数据库变更模板

涉及新增字段或表时，先补充：

```text
表名：
字段：
类型：
是否可空：
默认值：
索引：
旧数据兼容策略：
是否影响查询性能：
```

Codex 处理数据库变更时必须：

- 查找现有 SQL 迁移脚本风格。
- 同步修改 Entity / Model / DAO。
- 检查分页查询、排序、过滤条件是否受影响。
- 避免直接写破坏性迁移。

## 8. 权限检查清单

每个后端接口都要确认：

- 系统管理员是否可访问？
- 租户管理员是否只能访问本租户数据？
- 客户用户是否只能访问分配给自己的数据？
- 是否需要校验 entity ownership？
- 是否会泄漏其他 tenant/customer 的数据？
- 前端按钮是否也需要隐藏或禁用？

提示词：

```text
请重点检查这个改动是否破坏 ThingsBoard 的 tenant/customer 权限边界。
```

## 9. 常用开发任务提示词

### 定位功能

```text
请省 token 定位这个功能在哪里实现。只用 rg 搜索和少量文件阅读，列出最相关的文件、类、组件、接口路径，不要改代码。
功能是：……
```

### 新增接口

```text
请新增一个后端接口，并接入前端调用。先定位相似接口，沿用现有 Controller/Service/DAO 风格，不新增无关抽象。
接口需求：……
```

### 新增页面

```text
请新增一个 PC 前端页面。先定位相似页面、路由、菜单、权限和 i18n，然后按现有风格实现。
页面需求：……
```

### 修改字段

```text
请为现有业务对象新增/修改字段。先定位前端 model、表单、详情页、后端 DTO/Entity/DAO、数据库迁移脚本，再给出影响范围。
字段需求：……
```

### 修 bug

```text
请按 bug 修复模式处理：先复现/定位可能原因，列出最小修改方案，再改代码。不要顺手重构。
现象：……
期望：……
```

## 10. 提交前验证清单

根据改动范围选择执行：

```text
前端:
  npm run lint
  npm run build
  npm test

后端:
  mvn test
  mvn -pl <module> test
  mvn -DskipTests package

联调:
  检查浏览器控制台
  检查接口 HTTP 状态码
  检查权限账号访问结果
  检查数据库迁移是否正常
```

Codex 完成修改后需要输出：

- 修改了哪些文件
- 前后端调用链
- 数据库是否变更
- 权限是否变更
- 已运行的验证命令
- 未验证的风险点

## 11. 禁止事项

除非明确要求，Codex 不应：

- 全仓库格式化。
- 升级 Angular、Spring、Maven、Node、npm 依赖。
- 删除或重命名 ThingsBoard 核心模块。
- 绕过权限校验。
- 直接修改构建产物。
- 大范围重构 shared/core/common 模块。
- 在没有迁移脚本的情况下修改数据库结构。
- 把业务逻辑堆进 Controller 或前端组件。

## 12. 每次任务开场固定语句

建议每次直接贴这个：

```text
请先阅读 docs/codex-context.md，并按 ThingsBoard 全栈二开省 token 模式处理。

本次需求：
……

要求：
1. 先定位相关文件，不要直接改；
2. 输出前端页面/API/后端 Controller/Service/DAO/DB 的调用链；
3. 说明准备修改哪些文件；
4. 保持现有风格，不做无关重构；
5. 我确认后再实现。
```

