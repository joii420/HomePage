# 实施计划书：SRM 寻源项目管理系统
234
**分支**：`001-srm-sourcing-fullspec` | **日期**：2026-03-20 | **规格文档**：[spec.md](./spec.md)

## 概述

构建完整的 SRM（供应商关系管理）寻源项目管理系统，涵盖采购需求、寻源项目、招投标、技术评审、报价竞价、供应商管理等 16 个业务模块。P1 阶段聚焦核心寻源链路：登录认证 → 采购需求管理 → 寻源项目管理 → 招投标管理 → 技术方案管理 → 供应商门户，外加必要的支撑模块（供应商管理基础、报价评审）。

## 技术背景

**语言/版本**：Java 17  
**主要依赖**：Quarkus 3.23.3、Hibernate ORM with Panache、RESTEasy Reactive、Camunda 7（REST）、MinIO SDK、SmallRye JWT  
**前端**：React 18 + Ant Design（antd）、Axios、React Router、Zustand/Redux  
**存储**：PostgreSQL 15、Flyway 数据库迁移、MinIO（对象存储）  
**测试**：JUnit 5 + Quarkus Test、REST Assured、Testcontainers（PostgreSQL）  
**目标平台**：Docker + Nginx（Linux 服务器）  
**项目类型**：Web 应用（前后端分离，单体后端）  
**性能目标**：列表接口 ≤1s、首屏加载 ≤3s、竞价实时延迟 ≤2s、100 并发用户  
**约束条件**：单体架构（禁止微服务）、跨模块仅 Service 层调用、Camunda 不操作业务库  
**规模范围**：1000+ 供应商、13 种用户角色、16 个业务模块（P1 含 6 个核心模块 + 支撑模块）

## 架构原则校验

*入口条件：必须在第 0 阶段研究前通过。第 1 阶段设计完成后重新校验。*

| 原则 | 校验标准 | 状态 |
|------|----------|------|
| I. 单体内聚 | 后端单体，前后端独立 Docker 容器 | ✅ 通过 |
| II. 严格模块边界 | 按模块包结构，跨模块仅 Service→Service | ✅ 通过 |
| III. 工作流委托 | 所有审批走 Camunda 7 REST 回调，不直驱流程 | ✅ 通过 |
| IV. 安全优先 | RBAC 三层模型，MinIO 代理访问，预签名 30 分钟 | ✅ 通过 |
| V. 可观测性与审计 | CDI Interceptor 统一日志，关键操作记录前后状态 | ✅ 通过 |
| VI. 分阶段交付 | P1：登录→需求→寻源→招投标→技术方案→供应商门户 | ✅ 通过 |
| VII. 简洁性优先 | 优先 Panache/RESTEasy 内置能力，YAGNI 原则 | ✅ 通过 |

## 项目目录结构

### 文档（本功能）

```text
specs/001-srm-sourcing-fullspec/
├── plan.md              # 本文件
├── research.md          # 第 0 阶段输出
├── data-model.md        # 第 1 阶段输出
├── quickstart.md        # 第 1 阶段输出
├── contracts/           # 第 1 阶段输出（REST API 契约）
└── tasks.md             # 第 2 阶段输出（/speckit.tasks）
```

### 源代码（仓库根目录）

```text
backend/
├── src/main/java/com/srm/
│   ├── common/                    # 公共基础设施
│   │   ├── entity/                # BaseEntity, AuditFields
│   │   ├── resource/              # 统一响应包装, 分页参数
│   │   ├── security/              # JWT, RBAC, PasswordUtil
│   │   ├── interceptor/           # AuditLogInterceptor (CDI)
│   │   ├── config/                # MinioConfig, CamundaConfig
│   │   └── exception/             # GlobalExceptionHandler
│   ├── auth/                      # 模块零: 登录认证
│   │   ├── entity/                # User, Role, Permission
│   │   ├── service/               # AuthService, UserService
│   │   └── resource/              # AuthResource, UserResource
│   ├── procurement/               # 模块一: 采购需求管理
│   │   ├── entity/                # PurchaseRequest, PurchaseRequestItem, ProcurementDemand
│   │   ├── service/               # PurchaseRequestService, DemandPoolService
│   │   └── resource/              # PurchaseRequestResource, DemandPoolResource
│   ├── sourcing/                  # 模块二: 寻源项目管理
│   │   ├── entity/                # SourcingProject, ProjectMember, Timeline, SupplierInvitation, Clarification
│   │   ├── service/               # SourcingProjectService, TimelineService, InvitationService
│   │   └── resource/              # SourcingProjectResource
│   ├── bidding/                   # 模块三: 招投标管理
│   │   ├── entity/                # BidOpening, BidEvaluation, BidAward, AwardNotice
│   │   ├── service/               # BidOpeningService, EvaluationService, AwardService
│   │   └── resource/              # BiddingResource
│   ├── quotation/                 # 模块四: 报价与竞价管理
│   │   ├── entity/                # QuotationReview, AuctionSession, AuctionBid, MultiRoundQuote
│   │   ├── service/               # QuotationService, AuctionService
│   │   └── resource/              # QuotationResource, AuctionResource
│   ├── techreview/                # 模块六: 技术方案管理
│   │   ├── entity/                # TechProposal, TechReview, Expert, ExpertReviewTask
│   │   ├── service/               # TechReviewService, ExpertService
│   │   └── resource/              # TechReviewResource, ExpertResource
│   ├── supplier/                  # 模块十: 供应商管理
│   │   ├── entity/                # Supplier, SupplierRegistration, Qualification, Classification, Blacklist
│   │   ├── service/               # SupplierService, RegistrationService
│   │   └── resource/              # SupplierResource, RegistrationResource
│   ├── portal/                    # 模块九: 供应商端门户
│   │   ├── service/               # PortalProjectService, PortalQuoteService
│   │   └── resource/              # PortalResource
│   ├── workflow/                  # Camunda 集成层
│   │   ├── service/               # WorkflowService（启动/查询流程）
│   │   └── resource/              # WorkflowCallbackResource（接收回调）
│   └── file/                      # 文件管理
│       ├── service/               # FileService（MinIO 操作）
│       └── resource/              # FileResource
├── src/main/resources/
│   ├── application.properties
│   ├── db/migration/              # Flyway SQL 脚本
│   └── META-INF/
└── src/test/java/com/srm/
    ├── auth/
    ├── procurement/
    ├── sourcing/
    └── ...

frontend/
├── src/
│   ├── api/                       # API 客户端（Axios）
│   ├── components/                # 通用组件
│   ├── layouts/                   # 布局组件（采购方/供应商门户）
│   ├── pages/
│   │   ├── auth/                  # 登录页
│   │   ├── procurement/           # 采购需求
│   │   ├── sourcing/              # 寻源项目
│   │   ├── bidding/               # 招投标
│   │   ├── techreview/            # 技术方案
│   │   ├── quotation/             # 报价竞价
│   │   └── portal/                # 供应商门户
│   ├── store/                     # 状态管理
│   ├── hooks/                     # 自定义 Hooks
│   └── utils/                     # 工具函数
├── public/
└── tests/
```

**目录结构说明**：Web 应用结构，前后端独立目录。后端按业务模块组织包（`com.srm.{module}`），每个模块含 entity / service / resource 三层。供应商门户（portal）复用后端 Service 层，仅独立 Resource 端点。

## 复杂度跟踪

> 无违规——架构与全部 7 条原则保持一致。

| 设计决策 | 理由 | 被否决的简单方案及原因 |
|----------|------|----------------------|
| Camunda 7 REST 集成 | 架构原则 III 要求工作流委托 | 内嵌审批逻辑无法灵活配置 11 种审批流程 |
| WebSocket + SSE + 轮询三级降级 | 竞价模块实时性要求 ≤2s | 仅轮询无法满足秒级推送需求 |

## P1 模块依赖图

```text
登录认证（auth）
  └── 所有模块均依赖

供应商管理（supplier）—— 基础 CRUD
  └── 寻源项目（sourcing）、供应商门户（portal）依赖

采购需求（procurement）
  └── 寻源项目（sourcing）依赖（需求池 → 创建项目）

寻源项目（sourcing）
  ├── 招投标（bidding）依赖
  ├── 技术方案（techreview）依赖
  └── 报价竞价（quotation）依赖

供应商门户（portal）
  └── 依赖 sourcing、quotation、techreview
```

## P1 与 P2/P3 边界

### P1 范围（本次实现）

| 模块 | 包含的用户故事 |
|------|--------------|
| 模块零：登录认证 | US-0.1, US-0.3 |
| 模块一：采购需求 | US-1.1, US-1.2 |
| 模块二：寻源项目 | US-2.1, US-2.2 |
| 模块三：招投标 | US-3.1, US-3.2, US-3.3, US-3.4 |
| 模块四：报价竞价 | US-4.1, US-4.2, US-4.4 |
| 模块六：技术方案 | US-6.1, US-6.3 |
| 模块九：供应商门户 | US-9.1, US-9.2, US-9.3 |
| 模块十：供应商管理 | US-10.0, US-10.1（基础 CRUD + 注册） |

### P2/P3 延期（预留接口桩）

- 模块五：保证金管理
- 模块七：预算管理
- 模块八：模板管理
- 模块十一～十五：合同 / 电子签章 / 档案 / 绩效 / 风险管理
- P1 模块内的 P2 故事：US-0.2, US-1.3, US-2.3～2.7, US-4.3, US-6.2, US-9.4～9.5, US-10.2～10.6

## 关键技术决策

### 1. 认证与会话管理

- 采用 JWT（SmallRye JWT）无状态认证
- 访问令牌有效期：30 分钟（与会话超时一致）
- 刷新令牌有效期：7 天（对应「记住我」功能）
- 登录失败锁定：基于数据库计数器，5 次失败 / 30 分钟锁定
- 独立登录端点：`/api/v1/auth/buyer/login`（采购方）、`/api/v1/auth/supplier/login`（供应商）

### 2. RBAC 权限模型

- 三表模型：`srm_auth_user`、`srm_auth_role`、`srm_auth_permission`
- 关联表：`srm_auth_user_role`（用户角色）、`srm_auth_role_permission`（角色权限）
- 权限格式：`{模块}:{资源}:{操作}`，例如 `sourcing:project:create`
- 菜单权限与操作权限分离
- 通过 JWT Claim 携带部门/组织单元实现数据隔离

### 3. Camunda 7 工作流集成

- Camunda 作为独立服务运行，使用独立数据库
- 业务系统 → Camunda：REST API 启动流程 / 查询任务
- Camunda → 业务系统：任务完成时发送 REST 回调
- 回调端点：`POST /api/v1/workflow/callback`
- 回调 Payload：`{ processInstanceId, businessKey, approvalResult, approverName, approvalComment, completedAt }`

### 4. 文件存储（MinIO）

- Bucket 命名：`srm-{env}`，例如 `srm-dev`、`srm-prod`
- 对象路径：`{bizType}/{bizId}/{timestamp}_{filename}`
- 上传：前端 Multipart → 后端 → MinIO
- 下载：后端代理（先鉴权）→ 从 MinIO 流式传输
- 预览：生成预签名 URL，有效期 30 分钟
- 清理：定时任务清理孤立文件（72 小时后）

### 5. 竞价实时通信（仅竞价模块）

- 主要方案：WebSocket（`/ws/auction/{sessionId}`）
- 降级方案一：SSE（`/api/v1/auction/{sessionId}/stream`）
- 降级方案二：轮询（`GET /api/v1/auction/{sessionId}/status`，3 秒间隔）
- 心跳检测：30 秒 ping/pong
- 断线重连：自动同步遗漏出价记录

### 6. API 接口规范

- 基础路径：`/api/v1/{模块}/{资源}`
- 统一响应结构：`{ "code": 200, "message": "success", "data": {...} }`
- 分页请求：`?page=1&size=10` → 响应：`{ "items": [], "total": 100, "page": 1, "size": 10 }`
- 错误响应：`{ "code": 40001, "message": "描述", "data": null }`
- 排序参数：`?sort=createdAt,desc`
- 过滤参数：按字段名传 Query 参数

### 7. 数据库规范

- 表名前缀：`srm_{模块缩写}_{实体}`，例如 `srm_src_project`、`srm_bid_evaluation`
- 模块缩写对照：auth, proc, src, bid, quot, tech, sup, portal, wf, file
- 标准字段（所有表）：`id`（UUID）、`created_at`、`updated_at`、`created_by`、`updated_by`
- 软删除：`deleted`（布尔值）+ `deleted_at`（时间戳）
- 枚举存储：字符串类型（非序数值）

## 补充说明

- P1 阶段需要供应商管理的基础 CRUD（注册、列表、详情），但不含画像、分类、黑名单等 P2 功能。
- 模板管理（模块八）在 P1 中暂用硬编码评分模型（5 维度固定权重），P2 再实现可视化模板设计器。
- 报价模板在 P1 中使用固定列结构，P2 再支持自定义列和计算公式。
