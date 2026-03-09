# Beego 代码库新人导览

> 面向第一次阅读 `github.com/astaxie/beego` 源码的同学，目标是快速建立“这是什么、从哪里入手、怎么持续深入”的心智模型。

## 1. 先看整体：这是一个“分层 + 模块化”的 Go Web 全家桶

仓库根目录按职责大致可分成 4 类：

1. **`server/`**：服务端能力，核心是 `server/web`（HTTP Web 框架）。
2. **`client/`**：客户端/基础能力，包括 `orm`、`cache`、`httplib`。
3. **`core/`**：跨领域通用能力，比如 `config`、`logs`、`validation`、`utils`、`bean`。
4. **`task/`**：定时任务模块（toolbox/task）。

此外还有：

- **`adapter/`**：兼容适配层（给旧路径/旧调用方式提供过渡）。
- **`test/`**：集成测试相关资源（如视图模板）。
- 根目录 `README.md` 给出项目定位、模块地图和快速启动方式。

这与 README 中的“四大组成”是一致的：Base modules、Task、Client、Server。

---

## 2. 每个目录该怎么理解（新人视角）

### 2.1 `server/web`：请求生命周期主战场

这是 Web 框架最核心的一层，建议优先熟悉：

- `server.go` / `beego.go`：应用启动入口与运行流程。
- `router.go` / `tree.go` / `namespace.go`：路由注册与匹配机制。
- `controller.go`：Controller 约定与请求处理。
- `context/`：请求上下文输入输出（`input.go`、`output.go`、`context.go`）。
- `filter.go` / `hooks.go`：过滤器与扩展点。
- `config.go`：Web 相关配置。

**为什么先看它**：
新人最容易通过“一个 HTTP 请求在框架里经历了什么”来建立全局认知。搞懂这条链路后，再看 ORM/日志/配置会更有锚点。

### 2.2 `client/orm`：数据访问与模型映射

典型关注点：

- 模型注册与元数据构建（`models*.go`）。
- 查询构造（`orm_queryset.go`、`orm_conds.go`、`orm_raw.go`）。
- 多数据库方言支持（`db_mysql.go`、`db_postgres.go`、`db_oracle.go` 等）。
- 数据库别名/连接管理（`db_alias.go`）。

**新人建议**：不要一开始就深挖所有方言实现，先抓“统一接口 + 方言分发”这条主线。

### 2.3 `core/`：基础设施与可复用能力

最常用的几个子模块：

- `core/config`：统一配置读取。
- `core/logs`：日志接口与多种 logger 适配。
- `core/validation`：参数校验能力。
- `core/utils`：常见工具函数。
- `core/admin`：运行时观测/管理相关能力。

**阅读策略**：先看对外入口，再看具体 provider/adapter 实现，避免陷入细节。

### 2.4 `adapter/`：兼容层

`adapter/` 下有 `cache`、`config`、`orm`、`session`、`toolbox` 等镜像式目录结构，通常用于兼容旧版本调用路径或 API。

**新人要点**：
- 新功能开发通常应优先看 `core/`、`client/`、`server/` 主路径。
- 维护历史代码时再回到 `adapter/` 处理兼容行为。

---

## 3. 除源码外，必须知道的“工程化信息”

1. **模块与版本**：`go.mod` 显示模块名是 `github.com/astaxie/beego`，并声明了 Go 版本与依赖范围。
2. **版本常量**：`build_info.go` 的 `VERSION`（当前仓库为 `2.0.0`）有助于定位文档和行为差异。
3. **贡献规范**：`CONTRIBUTING.md` 包含本地检查工具建议（如 goimports、staticcheck）与提交流程说明。
4. **子模块 README**：例如 `client/orm/README.md`、`core/logs/README.md`、`client/cache/README.md` 等，能快速获得模块级使用心智。

---

## 4. 建议的新人成长路径（按 2~4 周节奏）

### 第 1 周：建立请求链路认知

- 跑通最小 Web 示例（`web.Run()`）。
- 从 `server/web` 出发，画出一张“请求进入 -> 路由 -> Controller -> Response”流程图。
- 用断点/日志验证你画的关键节点。

### 第 2 周：补齐基础设施

- 阅读 `core/config` 和 `core/logs` 的入口与默认实现。
- 在 demo 中加入配置文件与结构化日志，理解框架如何把“配置 -> 行为”串起来。

### 第 3 周：连接数据层

- 用 `client/orm` 做一个最小 CRUD。
- 重点理解：模型注册、查询构造、连接别名。
- 知道什么情况下使用 raw SQL，什么情况下使用 QuerySet。

### 第 4 周：实战与贡献

- 选择一个小 bug（优先测试易复现的）。
- 先补测试再修复，按 `CONTRIBUTING.md` 建议做格式化和静态检查。
- 提交 PR，关注 reviewer 对设计边界与兼容性的反馈。

---

## 5. 新人常见误区

1. **一上来全量读完**：模块太多，容易“知道名词，不懂关系”。
2. **忽略测试文件**：很多行为细节其实在 `*_test.go` 里最清楚。
3. **混淆主路径与兼容路径**：把 `adapter/` 当主实现会绕远路。
4. **先钻底层再看入口**：正确顺序应是“入口 -> 流程 -> 扩展点 -> 细节实现”。

---

## 6. 给带教同学的落地建议

- 第一轮 code walkthrough 控制在 60~90 分钟，只讲主链路。
- 每次只给新人一个“可验证目标”（例如：增加一个路由并打出自定义日志）。
- 用“问题驱动阅读”替代“目录驱动阅读”，如：
  - “参数校验在哪一层触发？”
  - “session 从哪里注入到上下文？”
  - “ORM 是如何切换不同数据库方言的？”

当新人能回答这类问题并在代码中定位到位置，就说明已经从“会用框架”进入“理解框架”。
