# 任务清单：SRM 寻源项目a111111111111111111111111管理系统
3454441112222222233333
**输入文档**：`/specs/001-srm-sourcing-fullspeadsc/`
**前置依赖**：plan.mdasdasd、spec.md、data-model.md、contracts/、research.md、quickstart.md

**测试**：未明确要求测试任务，已省略。

**组织方式**：任务按模块分组，每个模块阶段开头附 API 接口清单。

---

## API 接口统计摘要

| 模块 | P1接口数 | P2/P3接口数 | 外部回调数 |
|------|---------|------------|-----------|
| 登录认证 | 5 | 1 | 0 |
| 采购需求管理 | 12 | 1 | 0 |
| 寻源项目管理 | 11 | 7 | 0 |
| 招投标管理 | 11 | 0 | 0 |
| 报价与竞价管理 | 9（+1 WebSocket） | 2 | 0 |
| 技术方案管理 | 11 | 4 | 0 |
| 供应商管理 | 8 | 6 | 0 |
| 供应商门户 | 8 | 3 | 0 |
| 工作流集成 | 2 | 0 | 1 |
| 文件管理 | 4 | 0 | 0 |
| （P2）保证金管理 | — | 8 | 0 |
| （P2）预算管理 | — | 10 | 0 |
| （P2）模板管理 | — | 12 | 0 |
| （P2）合同管理 | — | 18 | 0 |
| （P2）电子签章 | — | 8 | 1 |
| （P2）档案管理 | — | 10 | 0 |
| （P2）供应商绩效管理 | — | 10 | 0 |
| （P2）风险与治理中心 | — | 12 | 0 |
| **合计** | **81（+1 WS）** | **102** | **2** |

> P2/P3 模块的接口清单列出但全部标记为 ❌ 跳过，
> implement 阶段不生成对应代码，仅作范围记录。

---

## 任务格式：`[编号] [P?] [关联US] 描述`

- **[P]**：可并行执行（操作不同文件，无依赖关系）
- **[关联US]**：对应的用户故事编号（如 US0.1、US1.1）
- 描述中包含精确文件路径
- **接口范围** 字段标注每个任务涉及的接口

---

## 阶段一：初始化

**目标**：项目初始化与基础结构搭建

- [ ] T001 创建后端项目结构：`backend/` 含 Maven Wrapper、`pom.xml`（Quarkus 3.23.3、Java 17，依赖：quarkus-resteasy-reactive-jackson、quarkus-hibernate-orm-panache、quarkus-jdbc-postgresql、quarkus-flyway、quarkus-smallrye-jwt、quarkus-websockets-next、quarkus-mailer、MinIO SDK）
  - **接口范围**：本任务不涉及接口，仅创建项目骨架

- [ ] T002 [P] 创建前端项目结构：`frontend/` 含 React 18 + Vite、Ant Design、Axios、React Router、Zustand
  - **接口范围**：本任务不涉及接口，仅创建前端项目骨架

- [ ] T003 [P] 创建 `backend/src/main/resources/application.properties`，配置 PostgreSQL、MinIO、Camunda、JWT、Flyway（开发环境配置）
  - **接口范围**：本任务不涉及接口，仅配置文件

- [ ] T004 [P] 在项目根目录创建 `docker-compose.yml`，包含 PostgreSQL 15、MinIO、Camunda 7 服务（参照 quickstart.md）
  - **接口范围**：本任务不涉及接口，仅基础设施

- [ ] T005 [P] 创建 `.gitignore`，覆盖 Java/Maven + Node/React + IDE 文件
  - **接口范围**：本任务不涉及接口

---

## 阶段二：基础设施（必须优先完成）

**目标**：所有用户故事均依赖的核心基础设施

**⚠️ 关键**：本阶段完成前，不得开始任何用户故事的开发

### API 接口清单（基础设施）

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| POST | /api/v1/files/upload | 上传文件（multipart） | 全局 | ✅ P1 |
| GET | /api/v1/files/{id}/download | 下载文件（后端代理） | 全局 | ✅ P1 |
| GET | /api/v1/files/{id}/preview-url | 获取预签名预览URL（30分钟有效） | 全局 | ✅ P1 |
| DELETE | /api/v1/files/{id} | 删除文件 | 全局 | ✅ P1 |
| POST | /api/v1/workflow/callback | Camunda 流程回调接收端 | 全局 | ✅ P1 外部回调 |
| GET | /api/v1/workflow/tasks | 获取当前用户待办审批任务 | 全局 | ✅ P1 |
| GET | /api/v1/workflow/instances/{businessKey} | 查询业务单据流程状态 | 全局 | ✅ P1 |

### 任务列表

- [ ] T006 创建 BaseEntity，字段含 UUID 主键、created_at、updated_at、created_by、updated_by、deleted、deleted_at，路径：`backend/src/main/java/com/srm/common/entity/BaseEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 基类

- [ ] T007 [P] 创建 `ApiResponse<T>`，字段含 code、message、data，路径：`backend/src/main/java/com/srm/common/resource/ApiResponse.java`
  - **接口范围**：本任务不涉及接口，仅实现响应包装类

- [ ] T008 [P] 创建 PageRequest（page/size 校验：10/20/50）和 PageResult（items/total/page/size），路径：`backend/src/main/java/com/srm/common/resource/PageRequest.java` 和 `PageResult.java`
  - **接口范围**：本任务不涉及接口，仅实现分页参数类

- [ ] T009 [P] 使用 JAX-RS ExceptionMapper 创建 GlobalExceptionHandler，路径：`backend/src/main/java/com/srm/common/exception/GlobalExceptionHandler.java`
  - **接口范围**：本任务不涉及接口，仅实现全局异常处理

- [ ] T010 [P] 创建带错误码和消息的 BusinessException，路径：`backend/src/main/java/com/srm/common/exception/BusinessException.java`
  - **接口范围**：本任务不涉及接口，仅实现业务异常类

- [ ] T011 [P] 创建 CDI AuditLogInterceptor，含 @AuditLog 注解，路径：`backend/src/main/java/com/srm/common/interceptor/AuditLogInterceptor.java` 和 `AuditLog.java`
  - **接口范围**：本任务不涉及接口，仅实现审计日志拦截器

- [ ] T012 [P] 创建 AuditLogEntity（表名：srm_audit_log），路径：`backend/src/main/java/com/srm/common/entity/AuditLogEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T013 创建 Flyway 迁移脚本 `V1__baseline_common.sql`，建 srm_audit_log 表，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T014 [P] 创建 MinioConfig 和 FileService（上传、流式下载、预签名URL、删除），路径：`backend/src/main/java/com/srm/file/config/MinioConfig.java` 和 `backend/src/main/java/com/srm/file/service/FileService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T015 [P] 创建 FileAttachmentEntity（表名：srm_file_attachment），路径：`backend/src/main/java/com/srm/file/entity/FileAttachmentEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T016 创建 FileResource，实现文件上传/下载/预览URL/删除接口，路径：`backend/src/main/java/com/srm/file/resource/FileResource.java`
  - **接口范围**：
  - 本任务实现：`POST /api/v1/files/upload`、`GET /api/v1/files/{id}/download`、`GET /api/v1/files/{id}/preview-url`、`DELETE /api/v1/files/{id}`
  - 本任务跳过：无

- [ ] T017 创建 Flyway 迁移脚本 `V2__file_attachment.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T018 [P] 创建 WorkflowService，封装 Camunda 7 REST 操作（startProcess、queryTask、completeTask），路径：`backend/src/main/java/com/srm/workflow/service/WorkflowService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T019 [P] 创建 WorkflowCallbackResource（`POST /api/v1/workflow/callback`），按 businessKey 分发，路径：`backend/src/main/java/com/srm/workflow/resource/WorkflowCallbackResource.java`
  - **接口范围**：
  - 本任务实现：`POST /api/v1/workflow/callback`（外部回调接收端）、`GET /api/v1/workflow/tasks`、`GET /api/v1/workflow/instances/{businessKey}`
  - 本任务跳过：无

- [ ] T020 [P] 创建 WorkflowInstanceEntity（表名：srm_wf_instance），路径：`backend/src/main/java/com/srm/workflow/entity/WorkflowInstanceEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T021 创建 Flyway 迁移脚本 `V3__workflow_and_notification.sql`，建 srm_wf_instance、srm_notif_notification 表，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T022 [P] 创建 NotificationEntity 和 NotificationService（站内通知 + 邮件），路径：`backend/src/main/java/com/srm/common/entity/NotificationEntity.java` 和 `backend/src/main/java/com/srm/common/service/NotificationService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service/Entity 层

- [ ] T023 [P] 创建前端 Axios 客户端，含 JWT 拦截器（自动附加 Bearer Token，401 时自动刷新），路径：`frontend/src/api/client.ts`
  - **接口范围**：本任务不涉及接口，仅前端 HTTP 客户端

- [ ] T024 [P] 创建前端认证状态管理（Zustand：登录状态、Token、用户信息），路径：`frontend/src/store/authStore.ts`
  - **接口范围**：本任务不涉及接口，仅前端状态管理

- [ ] T025 [P] 创建 BuyerLayout 和 SupplierPortalLayout，路径：`frontend/src/layouts/BuyerLayout.tsx` 和 `SupplierPortalLayout.tsx`
  - **接口范围**：本任务不涉及接口，仅前端布局组件

- [ ] T026 [P] 创建前端路由配置，含受保护路由，路径：`frontend/src/router/index.tsx`
  - **接口范围**：本任务不涉及接口，仅前端路由

- [ ] T027 [P] 创建通用组件：PageHeader、StatusTag、StatCard、FilterBar、DataTable（antd Table + 分页 10/20/50），路径：`frontend/src/components/common/`
  - **接口范围**：本任务不涉及接口，仅前端通用组件

**检查点**：基础设施就绪，可开始用户故事开发

---

## 阶段三：登录与认证 — US-0.1 采购方登录、US-0.3 供应商登录（P1）🎯 最小可用版本

**目标**：采购方用户和供应商用户可通过各自登录页完成身份认证

**独立测试**：使用正确凭据登录，获取 JWT Token，访问受保护 API

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| POST | /api/v1/auth/buyer/login | 采购方用户登录（用户名+密码→Token） | US-0.1 | ✅ P1 |
| POST | /api/v1/auth/supplier/login | 供应商用户登录 | US-0.3 | ✅ P1 |
| POST | /api/v1/auth/refresh | 刷新 Access Token | US-0.1 | ✅ P1 |
| POST | /api/v1/auth/logout | 退出登录 | US-0.1 | ✅ P1 |
| GET | /api/v1/auth/me | 获取当前用户信息+角色+权限 | US-0.1 | ✅ P1 |
| POST | /api/v1/auth/forgot-password | 忘记密码（邮箱验证码重置） | US-0.2 | ❌ P2跳过 |

### 任务列表

- [ ] T028 [P] [US0.1] 创建 UserEntity（表名：srm_auth_user，字段：username、password_hash、email、display_name、department、org_unit、status、lock_until、login_fail_count），路径：`backend/src/main/java/com/srm/auth/entity/UserEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T029 [P] [US0.1] 创建 RoleEntity（表名：srm_auth_role，字段：code、name、description），路径：`backend/src/main/java/com/srm/auth/entity/RoleEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T030 [P] [US0.1] 创建 PermissionEntity（表名：srm_auth_permission，字段：code、name、type MENU/ACTION、parent_id），路径：`backend/src/main/java/com/srm/auth/entity/PermissionEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T031 [P] [US0.1] 创建 UserRoleEntity 和 RolePermissionEntity 关联表，路径：`backend/src/main/java/com/srm/auth/entity/UserRoleEntity.java` 和 `RolePermissionEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T032 [P] [US0.1] 创建 LoginAttemptEntity（表名：srm_auth_login_attempt），路径：`backend/src/main/java/com/srm/auth/entity/LoginAttemptEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T033 创建 Flyway 迁移脚本 `V4__auth_tables.sql`，建所有认证表 + 初始数据（管理员账号、13个预置角色、基础权限），路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T034 [US0.1] 创建 PasswordUtil（BCrypt 哈希/校验），路径：`backend/src/main/java/com/srm/common/security/PasswordUtil.java`
  - **接口范围**：本任务不涉及接口，仅实现工具类

- [ ] T035 [US0.1] 创建 JwtService（生成 AccessToken 30分钟、RefreshToken 7天、校验Token、提取Claims），路径：`backend/src/main/java/com/srm/common/security/JwtService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T036 [US0.1] 创建 AuthService（登录含锁定逻辑：5次失败锁定30分钟、验证凭据、记录登录尝试、刷新Token），路径：`backend/src/main/java/com/srm/auth/service/AuthService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T037 [US0.1] 创建 AuthResource，实现采购方登录、刷新Token、退出、获取当前用户接口，路径：`backend/src/main/java/com/srm/auth/resource/AuthResource.java`
  - **接口范围**：
  - 本任务实现：`POST /api/v1/auth/buyer/login`、`POST /api/v1/auth/refresh`、`POST /api/v1/auth/logout`、`GET /api/v1/auth/me`
  - 本任务跳过：`POST /api/v1/auth/forgot-password`（P2）

- [ ] T038 [US0.3] 在 AuthResource 中添加供应商登录接口，路径：`backend/src/main/java/com/srm/auth/resource/AuthResource.java`
  - **接口范围**：
  - 本任务实现：`POST /api/v1/auth/supplier/login`
  - 本任务跳过：无

- [ ] T039 [P] [US0.1] 创建采购方登录页（含用户名/密码、"记住我"、锁定错误提示），路径：`frontend/src/pages/auth/BuyerLoginPage.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/auth/buyer/login`

- [ ] T040 [P] [US0.3] 创建供应商登录页（邮箱/密码），路径：`frontend/src/pages/auth/SupplierLoginPage.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/auth/supplier/login`

- [ ] T041 [US0.1] 实现会话超时：Axios 拦截器监听 401 → 跳转登录页并保留来源URL，路径：`frontend/src/api/client.ts`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/auth/refresh`

**检查点**：用户可登录、获取 JWT Token、访问受保护 API

---

## 阶段四：供应商管理 — US-10.0 供应商注册、US-10.1 全生命周期管理（P1）

**目标**：供应商可自助注册，管理员可管理供应商基础信息

**独立测试**：供应商注册→审核→激活→管理员可查看/编辑供应商

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| POST | /api/v1/supplier/register | 供应商自助注册提交 | US-10.0 | ✅ P1 |
| GET | /api/v1/supplier/register/check-credit-code | 信用代码实时唯一性校验 | US-10.0 | ✅ P1 |
| GET | /api/v1/supplier/register/status | 查询注册申请状态 | US-10.0 | ✅ P1 |
| GET | /api/v1/supplier/suppliers | 供应商列表（筛选+分页） | US-10.1 | ✅ P1 |
| POST | /api/v1/supplier/suppliers | 管理员手动新增供应商 | US-10.1 | ✅ P1 |
| GET | /api/v1/supplier/suppliers/{id} | 供应商详情 | US-10.1 | ✅ P1 |
| PUT | /api/v1/supplier/suppliers/{id} | 更新供应商信息 | US-10.1 | ✅ P1 |
| GET | /api/v1/supplier/suppliers/statistics | 供应商统计卡片 | US-10.1 | ✅ P1 |
| DELETE | /api/v1/supplier/suppliers/{id} | 删除供应商 | US-10.1 | ❌ P2跳过 |
| GET | /api/v1/supplier/suppliers/{id}/profile | 供应商画像 | US-10.2 | ❌ P2跳过 |
| PATCH | /api/v1/supplier/suppliers/{id}/classification | 分类与标签管理 | US-10.3 | ❌ P2跳过 |
| POST | /api/v1/supplier/suppliers/{id}/blacklist | 加入/移出黑名单 | US-10.4 | ❌ P2跳过 |
| GET | /api/v1/supplier/potential | 潜在供应商列表 | US-10.5 | ❌ P3跳过 |
| POST | /api/v1/supplier/potential/{id}/convert | 潜在供应商转正 | US-10.5 | ❌ P3跳过 |

### 任务列表

- [ ] T042 [P] [US10.1] 创建 SupplierEntity（表名：srm_sup_supplier，字段：name、credit_code、enterprise_type、registered_capital、founded_date、size、contact_person、contact_phone、email、classification、label、level、compliance_status、status），路径：`backend/src/main/java/com/srm/supplier/entity/SupplierEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T043 [P] [US10.0] 创建 SupplierRegistrationEntity（表名：srm_sup_registration，字段：supplier_name、credit_code、enterprise_type、contact_person、contact_phone、email、password_hash、status、review_comment、reviewed_by、reviewed_at），路径：`backend/src/main/java/com/srm/supplier/entity/SupplierRegistrationEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T044 [P] [US10.1] 创建 SupplierQualificationEntity（表名：srm_sup_qualification，字段：supplier_id、cert_name、cert_number、issuing_authority、valid_until、attachment_id、status），路径：`backend/src/main/java/com/srm/supplier/entity/SupplierQualificationEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T045 创建 Flyway 迁移脚本 `V5__supplier_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T046 [US10.0] 创建 RegistrationService（提交注册含信用代码校验+待审核状态校验、审核通过→创建供应商+用户+发送激活邮件、审核拒绝→发送拒绝邮件），路径：`backend/src/main/java/com/srm/supplier/service/RegistrationService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T047 [US10.0] 创建 RegistrationResource，路径：`backend/src/main/java/com/srm/supplier/resource/RegistrationResource.java`
  - **接口范围**：
  - 本任务实现：`POST /api/v1/supplier/register`、`GET /api/v1/supplier/register/check-credit-code`、`GET /api/v1/supplier/register/status`
  - 本任务跳过：无

- [ ] T048 [US10.1] 创建 SupplierService（列表含筛选和分页、getById、创建、更新、获取统计），路径：`backend/src/main/java/com/srm/supplier/service/SupplierService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T049 [US10.1] 创建 SupplierResource，路径：`backend/src/main/java/com/srm/supplier/resource/SupplierResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/supplier/suppliers`、`POST /api/v1/supplier/suppliers`、`GET .../suppliers/{id}`、`PUT .../suppliers/{id}`、`GET .../suppliers/statistics`
  - 本任务跳过：`DELETE .../suppliers/{id}`（P2）、`GET .../profile`（P2）、`PATCH .../classification`（P2）、`POST .../blacklist`（P2）

- [ ] T050 [P] [US10.0] 创建供应商注册页（公开页面，无需登录），路径：`frontend/src/pages/portal/SupplierRegistrationPage.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/supplier/register`、`GET /api/v1/supplier/register/check-credit-code`

- [ ] T051 [P] [US10.1] 创建供应商列表页（含统计卡片、状态Tab、筛选栏、数据表格），路径：`frontend/src/pages/supplier/SupplierListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/supplier/suppliers`、`GET .../suppliers/statistics`

- [ ] T052 [P] [US10.1] 创建供应商详情抽屉（基础信息Tab），路径：`frontend/src/pages/supplier/SupplierDetailDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`GET .../suppliers/{id}`、`PUT .../suppliers/{id}`

- [ ] T053 [US10.1] 创建新增供应商抽屉，路径：`frontend/src/pages/supplier/AddSupplierDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/supplier/suppliers`

**检查点**：供应商注册和管理员 CRUD 功能可用

---

## 阶段五：采购需求管理 — US-1.1 创建采购申请、US-1.2 管理需求池（P1）

**目标**：采购专员创建采购申请并提交审批，在需求池中管理需求行项

**独立测试**：创建申请→草稿→提交审批→回调→需求进池→选需求创建项目

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/procurement/requests | 采购申请列表（筛选+分页） | US-1.1 | ✅ P1 |
| POST | /api/v1/procurement/requests | 新建采购申请 | US-1.1 | ✅ P1 |
| GET | /api/v1/procurement/requests/{id} | 采购申请详情 | US-1.1 | ✅ P1 |
| PUT | /api/v1/procurement/requests/{id} | 更新采购申请 | US-1.1 | ✅ P1 |
| DELETE | /api/v1/procurement/requests/{id} | 删除采购申请（软删除） | US-1.1 | ✅ P1 |
| POST | /api/v1/procurement/requests/{id}/submit | 提交审批 | US-1.1 | ✅ P1 |
| POST | /api/v1/procurement/requests/{id}/withdraw | 撤回审批 | US-1.1 | ✅ P1 |
| GET | /api/v1/procurement/demands | 需求池列表（筛选+分页） | US-1.2 | ✅ P1 |
| POST | /api/v1/procurement/demands/merge | 合并需求（需同一采购方式） | US-1.2 | ✅ P1 |
| POST | /api/v1/procurement/demands/{id}/split | 拆分需求（总量必须等于原值） | US-1.2 | ✅ P1 |
| POST | /api/v1/procurement/demands/cancel | 取消需求 | US-1.2 | ✅ P1 |
| POST | /api/v1/procurement/demands/create-project | 从需求池创建寻源项目 | US-1.2 | ✅ P1 |
| GET | /api/v1/procurement/requests/statistics | 申请状态统计卡片 | US-1.3 | ❌ P3跳过 |

### 任务列表

- [ ] T054 [P] [US1.1] 创建 PurchaseRequestEntity（表名：srm_proc_purchase_request，字段：request_number、org_unit、department、applicant、apply_date、source_system、category、procurement_method、amount、currency、description、status），路径：`backend/src/main/java/com/srm/procurement/entity/PurchaseRequestEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T055 [P] [US1.1] 创建 PurchaseRequestItemEntity（表名：srm_proc_purchase_request_item），路径：`backend/src/main/java/com/srm/procurement/entity/PurchaseRequestItemEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T056 [P] [US1.2] 创建 ProcurementDemandEntity（表名：srm_proc_demand，字段：request_item_id、material_name、specification、quantity、unit、procurement_method、department、budget_amount、status、locked_by_project_id），路径：`backend/src/main/java/com/srm/procurement/entity/ProcurementDemandEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T057 创建 Flyway 迁移脚本 `V6__procurement_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T058 [US1.1] 创建 PurchaseRequestService（创建、更新、保存草稿、提交审批→Camunda、撤回、软删除、处理审批回调、获取统计），路径：`backend/src/main/java/com/srm/procurement/service/PurchaseRequestService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T059 [US1.1] 创建 PurchaseRequestResource，路径：`backend/src/main/java/com/srm/procurement/resource/PurchaseRequestResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/procurement/requests`、`POST .../requests`、`GET .../requests/{id}`、`PUT .../requests/{id}`、`DELETE .../requests/{id}`、`POST .../requests/{id}/submit`、`POST .../requests/{id}/withdraw`
  - 本任务跳过：`GET .../requests/statistics`（P3）

- [ ] T060 [US1.2] 创建 DemandPoolService（列表、合并校验、拆分校验、取消、创建寻源项目→锁定需求），路径：`backend/src/main/java/com/srm/procurement/service/DemandPoolService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T061 [US1.2] 创建 DemandPoolResource，路径：`backend/src/main/java/com/srm/procurement/resource/DemandPoolResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/procurement/demands`、`POST .../demands/merge`、`POST .../demands/{id}/split`、`POST .../demands/cancel`、`POST .../demands/create-project`
  - 本任务跳过：无

- [ ] T062 [P] [US1.1] 创建采购申请列表页（含5个统计卡片、筛选栏、数据表格），路径：`frontend/src/pages/procurement/PurchaseRequestListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/procurement/requests`

- [ ] T063 [P] [US1.1] 创建采购申请抽屉（1000px，4个Tab），路径：`frontend/src/pages/procurement/PurchaseRequestDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/procurement/requests`、`PUT .../requests/{id}`、`POST .../requests/{id}/submit`

- [ ] T064 [P] [US1.2] 创建采购需求池页面（含批量操作），路径：`frontend/src/pages/procurement/DemandPoolPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/procurement/demands`、`POST .../demands/merge`、`POST .../demands/cancel`、`POST .../demands/create-project`

- [ ] T065 [US1.2] 创建需求拆分弹窗，路径：`frontend/src/pages/procurement/SplitDemandModal.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/procurement/demands/{id}/split`

**检查点**：采购申请全生命周期和需求池管理功能可用

---

## 阶段六：寻源项目管理 — US-2.1 创建项目、US-2.2 管理全流程（P1）

**目标**：采购管理员从需求池创建寻源项目并管理全流程

**独立测试**：选需求→5步向导创建→发布审批→进行中→管理各Tab

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/sourcing/projects | 寻源项目列表（状态Tab+筛选） | US-2.1 | ✅ P1 |
| POST | /api/v1/sourcing/projects | 五步向导创建寻源项目 | US-2.1 | ✅ P1 |
| GET | /api/v1/sourcing/projects/{id} | 项目详情（含全部Tab数据） | US-2.2 | ✅ P1 |
| PUT | /api/v1/sourcing/projects/{id} | 更新项目信息 | US-2.2 | ✅ P1 |
| POST | /api/v1/sourcing/projects/{id}/publish | 发布项目（触发审批） | US-2.2 | ✅ P1 |
| GET | /api/v1/sourcing/projects/{id}/members | 获取项目成员列表 | US-2.1 | ✅ P1 |
| POST | /api/v1/sourcing/projects/{id}/members | 添加项目成员 | US-2.1 | ✅ P1 |
| DELETE | /api/v1/sourcing/projects/{id}/members/{userId} | 移除项目成员 | US-2.2 | ✅ P1 |
| GET | /api/v1/sourcing/projects/{id}/suppliers | 获取项目供应商列表 | US-2.1 | ✅ P1 |
| POST | /api/v1/sourcing/projects/{id}/suppliers/invite | 邀请供应商 | US-2.1 | ✅ P1 |
| PUT | /api/v1/sourcing/projects/{id}/timeline | 配置项目时间轴 | US-2.1 | ✅ P1 |
| DELETE | /api/v1/sourcing/projects/{id}/suppliers/{supplierId} | 移除已邀请供应商 | US-2.3 | ❌ P2跳过 |
| GET | /api/v1/sourcing/projects/{id}/clarifications | 澄清记录列表 | US-2.5 | ❌ P2跳过 |
| POST | /api/v1/sourcing/projects/{id}/clarifications | 提交澄清问题 | US-2.5 | ❌ P2跳过 |
| POST | /api/v1/sourcing/projects/{id}/clarifications/{cid}/reply | 回复澄清 | US-2.5 | ❌ P2跳过 |
| POST | /api/v1/sourcing/projects/{id}/announcement | 发布招标公告 | US-2.6 | ❌ P2跳过 |
| POST | /api/v1/sourcing/projects/{id}/result-publicity | 发布结果公示 | US-2.6 | ❌ P2跳过 |
| POST | /api/v1/sourcing/projects/{id}/cancel-bid | 废标处理 | US-2.7 | ❌ P2跳过 |

### 任务列表

- [ ] T066 [P] [US2.1] 创建 SourcingProjectEntity（表名：srm_src_project，字段：project_number、title、org_id、country、dev_type、biz_type、target_type、award_method、eval_method、allow_partial_quote、target_supplier_count、department、project_leader、budget、procurement_method、need_tech_review、need_auction、show_score_weight、status），路径：`backend/src/main/java/com/srm/sourcing/entity/SourcingProjectEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T067 [P] [US2.1] 创建 ProjectMemberEntity（表名：srm_src_project_member），路径：`backend/src/main/java/com/srm/sourcing/entity/ProjectMemberEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T068 [P] [US2.1] 创建 TimelineNodeEntity（表名：srm_src_timeline_node，字段：project_id、name、node_type FIXED/OPTIONAL、sort_order、start_time、end_time、allow_extension、is_draggable），路径：`backend/src/main/java/com/srm/sourcing/entity/TimelineNodeEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T069 [P] [US2.2] 创建 SupplierInvitationEntity（表名：srm_src_supplier_invitation，字段：project_id、supplier_id、status、invited_at、responded_at），路径：`backend/src/main/java/com/srm/sourcing/entity/SupplierInvitationEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T070 创建 Flyway 迁移脚本 `V7__sourcing_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T071 [US2.1] 创建 SourcingProjectService（从需求创建、更新、发布→Camunda、处理发布回调、处理驳回回调、列表、获取详情），路径：`backend/src/main/java/com/srm/sourcing/service/SourcingProjectService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T072 [US2.2] 创建 InvitationService 和 TimelineService，路径：`backend/src/main/java/com/srm/sourcing/service/InvitationService.java` 和 `TimelineService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T073 [US2.1] 创建 SourcingProjectResource，实现所有 P1 接口，路径：`backend/src/main/java/com/srm/sourcing/resource/SourcingProjectResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/sourcing/projects`、`POST .../projects`、`GET .../projects/{id}`、`PUT .../projects/{id}`、`POST .../projects/{id}/publish`、`GET .../projects/{id}/members`、`POST .../projects/{id}/members`、`DELETE .../projects/{id}/members/{userId}`、`GET .../projects/{id}/suppliers`、`POST .../projects/{id}/suppliers/invite`、`PUT .../projects/{id}/timeline`
  - 本任务跳过：澄清记录（P2）、招标公告（P2）、结果公示（P2）、废标（P2）

- [ ] T074 [P] [US2.1] 创建寻源项目列表页（含状态Tab、筛选栏），路径：`frontend/src/pages/sourcing/SourcingProjectListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/sourcing/projects`

- [ ] T075 [US2.1] 创建项目创建向导（5步），路径：`frontend/src/pages/sourcing/CreateProjectWizard.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/sourcing/projects`、`POST .../projects/{id}/suppliers/invite`、`PUT .../projects/{id}/timeline`、`POST .../projects/{id}/members`

- [ ] T076 [US2.2] 创建项目详情抽屉（7个Tab），路径：`frontend/src/pages/sourcing/ProjectDetailDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`GET .../projects/{id}`、`PUT .../projects/{id}`、`POST .../projects/{id}/publish`、`GET .../projects/{id}/members`、`GET .../projects/{id}/suppliers`

- [ ] T077 [P] [US2.1] 创建供应商选择抽屉（560px，按行业/资质筛选），路径：`frontend/src/pages/sourcing/SupplierSelectDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/supplier/suppliers`（跨模块调用供应商列表）

- [ ] T078 [P] [US2.1] 创建时间轴管理组件（固定节点+可拖拽可选节点），路径：`frontend/src/pages/sourcing/TimelineManager.tsx`
  - **接口范围**：
  - 本任务调用：`PUT /api/v1/sourcing/projects/{id}/timeline`

**检查点**：从需求池到已发布寻源项目的完整流程可用

---

## 阶段七：技术方案管理 — US-6.1 技术方案评审、US-6.3 分配评审专家（P1）

**目标**：采购管理员分配评审专家，专家对技术方案进行5维度加权评审

**独立测试**：分配专家→发送通知→评审打分→加权汇总

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/techreview/reviews | 技术评审列表 | US-6.1 | ✅ P1 |
| GET | /api/v1/techreview/reviews/{id} | 评审详情（含各专家评分） | US-6.1 | ✅ P1 |
| POST | /api/v1/techreview/reviews/{id}/submit-score | 专家提交评分 | US-6.1 | ✅ P1 |
| POST | /api/v1/techreview/reviews/{id}/reject-score | 驳回专家评分（打回重评） | US-6.1 | ✅ P1 |
| POST | /api/v1/techreview/reviews/{id}/assign | 分配评审专家 | US-6.3 | ✅ P1 |
| POST | /api/v1/techreview/reviews/{id}/reassign | 替换评审专家 | US-6.3 | ✅ P1 |
| GET | /api/v1/techreview/experts | 专家库列表 | US-6.3 | ✅ P1 |
| POST | /api/v1/techreview/experts | 新增专家 | US-6.3 | ✅ P1 |
| GET | /api/v1/techreview/experts/{id} | 专家详情 | US-6.3 | ✅ P1 |
| PUT | /api/v1/techreview/experts/{id} | 更新专家信息 | US-6.3 | ✅ P1 |
| GET | /api/v1/techreview/experts/{id}/assignments | 专家的评审任务列表 | US-6.3 | ✅ P1 |
| DELETE | /api/v1/techreview/experts/{id} | 删除专家 | US-6.2 | ❌ P2跳过 |
| PATCH | /api/v1/techreview/experts/{id}/status | 启用/停用专家 | US-6.2 | ❌ P2跳过 |
| GET | /api/v1/techreview/experts/{id}/evaluations | 专家历史评价 | US-6.2 | ❌ P2跳过 |
| GET | /api/v1/techreview/experts/{id}/certificates | 专家资质证书 | US-6.2 | ❌ P2跳过 |

### 任务列表

- [ ] T079 [P] [US6.1] 创建 ExpertEntity（表名：srm_tech_expert），路径：`backend/src/main/java/com/srm/techreview/entity/ExpertEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T080 [P] [US6.1] 创建 ExpertCertificateEntity（表名：srm_tech_expert_certificate），路径：`backend/src/main/java/com/srm/techreview/entity/ExpertCertificateEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T081 [P] [US6.1] 创建 TechProposalEntity（表名：srm_tech_proposal，字段：project_id、supplier_id、content_json、status），路径：`backend/src/main/java/com/srm/techreview/entity/TechProposalEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T082 [P] [US6.1] 创建 TechReviewEntity（表名：srm_tech_review，字段：project_id、supplier_id、expert_id、5个维度评分、weighted_total、comment、is_eliminated、elimination_reason、status），路径：`backend/src/main/java/com/srm/techreview/entity/TechReviewEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T083 [P] [US6.3] 创建 ExpertReviewTaskEntity（表名：srm_tech_expert_review_task），路径：`backend/src/main/java/com/srm/techreview/entity/ExpertReviewTaskEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T084 创建 Flyway 迁移脚本 `V8__techreview_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T085 [US6.1] 创建 TechReviewService（提交评分含5维度加权计算 25%+20%+20%+20%+15%、驳回评分、获取平均分、淘汰供应商），路径：`backend/src/main/java/com/srm/techreview/service/TechReviewService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T086 [US6.3] 创建 ExpertService（CRUD、分配到项目→创建任务+发送通知、替换专家、48小时催办提醒），路径：`backend/src/main/java/com/srm/techreview/service/ExpertService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T087 [US6.1] 创建 TechReviewResource，路径：`backend/src/main/java/com/srm/techreview/resource/TechReviewResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/techreview/reviews`、`GET .../reviews/{id}`、`POST .../reviews/{id}/submit-score`、`POST .../reviews/{id}/reject-score`、`POST .../reviews/{id}/assign`、`POST .../reviews/{id}/reassign`
  - 本任务跳过：无

- [ ] T088 [US6.3] 创建 ExpertResource，路径：`backend/src/main/java/com/srm/techreview/resource/ExpertResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/techreview/experts`、`POST .../experts`、`GET .../experts/{id}`、`PUT .../experts/{id}`、`GET .../experts/{id}/assignments`
  - 本任务跳过：`DELETE .../experts/{id}`（P2）、`PATCH .../experts/{id}/status`（P2）、`GET .../experts/{id}/evaluations`（P2）、`GET .../experts/{id}/certificates`（P2）

- [ ] T089 [P] [US6.3] 创建专家列表页（含筛选、统计），路径：`frontend/src/pages/techreview/ExpertListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/techreview/experts`

- [ ] T090 [P] [US6.3] 创建新增/编辑专家抽屉，路径：`frontend/src/pages/techreview/ExpertDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/techreview/experts`、`PUT .../experts/{id}`

- [ ] T091 [US6.3] 创建分配专家抽屉（含截止时间设置），路径：`frontend/src/pages/techreview/AssignExpertDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/techreview/reviews/{id}/assign`、`GET /api/v1/techreview/experts`

- [ ] T092 [P] [US6.1] 创建技术评审列表页，路径：`frontend/src/pages/techreview/TechReviewListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/techreview/reviews`

**检查点**：专家分配和技术方案评审全流程可用

---

## 阶段八：报价与竞价管理 — US-4.1 报价评审、US-4.2 电子竞价（P1）

**目标**：报价横向对比评审，以及实时电子竞价（WebSocket）

**独立测试**：多轮报价对比→综合评分→WebSocket实时竞价→手动/自动结束

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/quotation/reviews | 报价评审列表 | US-4.1 | ✅ P1 |
| GET | /api/v1/quotation/reviews/{id} | 报价评审详情（含横向对比） | US-4.1 | ✅ P1 |
| POST | /api/v1/quotation/reviews/{id}/submit | 提交评审结果 | US-4.1 | ✅ P1 |
| GET | /api/v1/quotation/auctions | 竞价列表 | US-4.2 | ✅ P1 |
| GET | /api/v1/quotation/auctions/{id} | 竞价详情 | US-4.2 | ✅ P1 |
| POST | /api/v1/quotation/auctions/{id}/start | 开始竞价 | US-4.2 | ✅ P1 |
| POST | /api/v1/quotation/auctions/{id}/end | 结束竞价 | US-4.2 | ✅ P1 |
| GET | /api/v1/quotation/auctions/{id}/stream | 竞价实时数据（SSE降级） | US-4.2 | ✅ P1 |
| POST | /api/v1/quotation/auctions/{id}/bid | 提交出价 | US-4.2 | ✅ P1 |
| WS | /ws/auction/{sessionId} | 竞价 WebSocket 实时连接 | US-4.2 | ✅ P1 |
| GET | /api/v1/quotation/multi-round | 多轮报价列表 | US-4.3 | ❌ P2跳过 |
| POST | /api/v1/quotation/multi-round/{id}/new-round | 发起新一轮报价 | US-4.3 | ❌ P2跳过 |

### 任务列表

- [ ] T093 [P] [US4.1] 创建 QuoteEntity（表名：srm_quot_quote，字段：project_id、supplier_id、round、total_amount、currency、status、submitted_at），路径：`backend/src/main/java/com/srm/quotation/entity/QuoteEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T094 [P] [US4.1] 创建 QuoteItemEntity（表名：srm_quot_quote_item，字段：quote_id、line_number、material_name、quantity、unit_price、discount、discounted_price、total），路径：`backend/src/main/java/com/srm/quotation/entity/QuoteItemEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T095 [P] [US4.2] 创建 AuctionSessionEntity（表名：srm_quot_auction_session），路径：`backend/src/main/java/com/srm/quotation/entity/AuctionSessionEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T096 [P] [US4.2] 创建 AuctionBidEntity（表名：srm_quot_auction_bid），路径：`backend/src/main/java/com/srm/quotation/entity/AuctionBidEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T097 [P] [US4.4] 创建 AuctionRuleEntity（表名：srm_quot_auction_rule，字段：min_decrement、min_increment、bid_interval_seconds、extension_trigger_minutes、extension_duration_minutes、max_extensions、allow_manual_end），路径：`backend/src/main/java/com/srm/quotation/entity/AuctionRuleEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T098 创建 Flyway 迁移脚本 `V9__quotation_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T099 [US4.1] 创建 QuotationService（获取对比详情含最高/最低价高亮、提交评审结果），路径：`backend/src/main/java/com/srm/quotation/service/QuotationService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T100 [US4.2] 创建 AuctionService（创建场次、开始、提交出价含规则校验、计算排名、处理延时、结束），路径：`backend/src/main/java/com/srm/quotation/service/AuctionService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T101 [US4.1] 创建 QuotationResource，路径：`backend/src/main/java/com/srm/quotation/resource/QuotationResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/quotation/reviews`、`GET .../reviews/{id}`、`POST .../reviews/{id}/submit`
  - 本任务跳过：`GET .../multi-round`（P2）、`POST .../multi-round/{id}/new-round`（P2）

- [ ] T102 [US4.2] 创建 AuctionResource，含竞价场次 CRUD、开始、结束、出价、SSE流接口，路径：`backend/src/main/java/com/srm/quotation/resource/AuctionResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/quotation/auctions`、`GET .../auctions/{id}`、`POST .../auctions/{id}/start`、`POST .../auctions/{id}/end`、`GET .../auctions/{id}/stream`（SSE）、`POST .../auctions/{id}/bid`
  - 本任务跳过：无

- [ ] T103 [US4.2] 创建竞价 WebSocket 端点，含30秒心跳、广播推送、断线数据同步，路径：`backend/src/main/java/com/srm/quotation/resource/AuctionWebSocket.java`
  - **接口范围**：
  - 本任务实现：`WS /ws/auction/{sessionId}`
  - 本任务跳过：无

- [ ] T104 [P] [US4.1] 创建报价评审列表页，路径：`frontend/src/pages/quotation/QuotationReviewListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/quotation/reviews`

- [ ] T105 [US4.1] 创建报价评审详情页（5步流程、对比表含最低绿色/最高红色高亮），路径：`frontend/src/pages/quotation/QuotationReviewDetail.tsx`
  - **接口范围**：
  - 本任务调用：`GET .../reviews/{id}`、`POST .../reviews/{id}/submit`

- [ ] T106 [P] [US4.2] 创建竞价列表页，路径：`frontend/src/pages/quotation/AuctionListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/quotation/auctions`

- [ ] T107 [US4.2] 创建竞价实时详情页（6个信息卡片、实时排名、价格趋势图、WebSocket+SSE+轮询降级），路径：`frontend/src/pages/quotation/AuctionLiveDetail.tsx`
  - **接口范围**：
  - 本任务调用：`GET .../auctions/{id}`、`WS /ws/auction/{sessionId}`、`GET .../auctions/{id}/stream`（SSE降级）、`POST .../auctions/{id}/bid`（轮询降级）

- [ ] T108 [US4.4] 创建竞价规则配置表单，路径：`frontend/src/pages/quotation/AuctionRuleConfig.tsx`
  - **接口范围**：本任务不涉及独立接口，表单数据作为创建竞价的一部分提交

**检查点**：报价评审和实时竞价含实时更新功能可用

---

## 阶段九：招投标管理 — US-3.1 开标、US-3.2 评标、US-3.3 定标、US-3.4 中标通知书（P1）

**目标**：完成开标→评标→定标→中标通知书全流程

**独立测试**：开标汇总→综合评分（技术60%+商务40%）→定标审批→生成PDF通知书→供应商确认

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/bidding/openings | 开标列表 | US-3.1 | ✅ P1 |
| GET | /api/v1/bidding/openings/{id} | 开标详情（供应商报价汇总） | US-3.1 | ✅ P1 |
| POST | /api/v1/bidding/openings/{id}/open | 执行开标 | US-3.1 | ✅ P1 |
| GET | /api/v1/bidding/evaluations | 评标列表 | US-3.2 | ✅ P1 |
| GET | /api/v1/bidding/evaluations/{id} | 评标详情（含评分排名） | US-3.2 | ✅ P1 |
| POST | /api/v1/bidding/evaluations/{id}/submit | 提交评标结果 | US-3.2 | ✅ P1 |
| GET | /api/v1/bidding/awards | 定标列表 | US-3.3 | ✅ P1 |
| GET | /api/v1/bidding/awards/{id} | 定标详情 | US-3.3 | ✅ P1 |
| POST | /api/v1/bidding/awards/{id}/submit-approval | 提交定标审批 | US-3.3 | ✅ P1 |
| GET | /api/v1/bidding/awards/{id}/notice | 中标通知书详情 | US-3.4 | ✅ P1 |
| POST | /api/v1/bidding/awards/{id}/notice/send | 发送中标通知书 | US-3.4 | ✅ P1 |

### 任务列表

- [ ] T109 [P] [US3.1] 创建 BidOpeningEntity（表名：srm_bid_opening），路径：`backend/src/main/java/com/srm/bidding/entity/BidOpeningEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T110 [P] [US3.2] 创建 BidEvaluationEntity（表名：srm_bid_evaluation，字段：project_id、supplier_id、tech_score、commercial_score、composite_score、tech_weight=60、commercial_weight=40、rank、status），路径：`backend/src/main/java/com/srm/bidding/entity/BidEvaluationEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T111 [P] [US3.3] 创建 BidAwardEntity（表名：srm_bid_award），路径：`backend/src/main/java/com/srm/bidding/entity/BidAwardEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T112 [P] [US3.4] 创建 AwardNoticeEntity（表名：srm_bid_award_notice，字段：project_id、award_id、supplier_id、amount、price_validity_days=90、confirm_deadline、supplier_confirmed、notice_pdf_file_id），路径：`backend/src/main/java/com/srm/bidding/entity/AwardNoticeEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T113 创建 Flyway 迁移脚本 `V10__bidding_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T114 [US3.1] 创建 BidOpeningService（获取详情含供应商报价汇总+最高/最低价、标记已开标），路径：`backend/src/main/java/com/srm/bidding/service/BidOpeningService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T115 [US3.2] 创建 EvaluationService（计算综合评分：技术×0.6+商务×0.4、获取排名）—— 调用 TechReviewService 和 QuotationService，路径：`backend/src/main/java/com/srm/bidding/service/EvaluationService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层（跨模块 Service→Service 调用）

- [ ] T116 [US3.3] 创建 AwardService（提交审批→Camunda、处理回调），路径：`backend/src/main/java/com/srm/bidding/service/AwardService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T117 [US3.4] 创建 AwardNoticeService（生成PDF、发送通知书、供应商确认、处理超时），路径：`backend/src/main/java/com/srm/bidding/service/AwardNoticeService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T118 [US3.1] 创建 BiddingResource，实现所有 P1 招投标接口，路径：`backend/src/main/java/com/srm/bidding/resource/BiddingResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/bidding/openings`、`GET .../openings/{id}`、`POST .../openings/{id}/open`、`GET .../evaluations`、`GET .../evaluations/{id}`、`POST .../evaluations/{id}/submit`、`GET .../awards`、`GET .../awards/{id}`、`POST .../awards/{id}/submit-approval`、`GET .../awards/{id}/notice`、`POST .../awards/{id}/notice/send`
  - 本任务跳过：无

- [ ] T119 [P] [US3.1] 创建开标列表页和开标详情页（汇总卡片、报价表含高亮），路径：`frontend/src/pages/bidding/BidOpeningListPage.tsx` 和 `BidOpeningDetail.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/bidding/openings`、`GET .../openings/{id}`

- [ ] T120 [P] [US3.2] 创建评标列表页和评标详情页（4个Tab：技术/商务/综合评分/评标报告），路径：`frontend/src/pages/bidding/BidEvaluationListPage.tsx` 和 `BidEvaluationDetail.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/bidding/evaluations`、`GET .../evaluations/{id}`

- [ ] T121 [P] [US3.3] 创建定标列表页和定标详情页（候选人排名、提交审批），路径：`frontend/src/pages/bidding/BidAwardListPage.tsx` 和 `BidAwardDetail.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/bidding/awards`、`GET .../awards/{id}`、`POST .../awards/{id}/submit-approval`

- [ ] T122 [US3.4] 创建中标通知书详情和供应商确认页面，路径：`frontend/src/pages/bidding/AwardNoticePage.tsx`
  - **接口范围**：
  - 本任务调用：`GET .../awards/{id}/notice`、`POST .../awards/{id}/notice/send`

**检查点**：开标→评标→定标→中标通知书全流程可用

---

## 阶段十：供应商门户 — US-9.1 报名、US-9.2 提交技术方案、US-9.3 报价（P1）

**目标**：供应商可通过门户完成报名、技术方案提交和在线报价

**独立测试**：供应商登录→查看项目→报名→提交技术方案→填写报价→提交

### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/portal/projects | 门户项目列表（按状态Tab筛选） | US-9.1 | ✅ P1 |
| POST | /api/v1/portal/projects/{id}/enroll | 报名参与项目 | US-9.1 | ✅ P1 |
| POST | /api/v1/portal/projects/{id}/decline | 拒绝参与项目 | US-9.1 | ✅ P1 |
| POST | /api/v1/portal/tech-proposals | 提交技术方案 | US-9.2 | ✅ P1 |
| PUT | /api/v1/portal/tech-proposals/{id} | 更新技术方案（保存草稿） | US-9.2 | ✅ P1 |
| POST | /api/v1/portal/tech-proposals/{id}/submit | 正式提交技术方案 | US-9.2 | ✅ P1 |
| POST | /api/v1/portal/quotes | 提交报价 | US-9.3 | ✅ P1 |
| POST | /api/v1/portal/quotes/import | Excel导入报价 | US-9.3 | ✅ P1 |
| GET | /api/v1/portal/projects/{id}/award-result | 查看定标结果 | US-9.4 | ❌ P2跳过 |
| POST | /api/v1/portal/quotes/{id}/withdraw | 撤回报价 | US-9.5 | ❌ P2跳过 |
| POST | /api/v1/portal/quotes/{id}/resubmit | 重新提交报价 | US-9.5 | ❌ P2跳过 |

### 任务列表

- [ ] T123 [P] [US9.1] 创建 PortalProjectView（视图聚合：项目信息+供应商邀请状态），路径：`backend/src/main/java/com/srm/portal/service/PortalProjectService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T124 [P] [US9.3] 创建 PortalQuoteEntity（表名：srm_portal_quote，关联 srm_quot_quote），路径：`backend/src/main/java/com/srm/portal/entity/PortalQuoteEntity.java`
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T125 创建 Flyway 迁移脚本 `V14__portal_tables.sql`，路径：`backend/src/main/resources/db/migration/`
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T126 [US9.1] 创建 PortalResource，实现所有 P1 门户接口，路径：`backend/src/main/java/com/srm/portal/resource/PortalResource.java`
  - **接口范围**：
  - 本任务实现：`GET /api/v1/portal/projects`、`POST .../projects/{id}/enroll`、`POST .../projects/{id}/decline`、`POST /api/v1/portal/tech-proposals`、`PUT .../tech-proposals/{id}`、`POST .../tech-proposals/{id}/submit`、`POST /api/v1/portal/quotes`、`POST .../quotes/import`
  - 本任务跳过：`GET .../projects/{id}/award-result`（P2）、`POST .../quotes/{id}/withdraw`（P2）、`POST .../quotes/{id}/resubmit`（P2）

- [ ] T127 [P] [US9.1] 创建供应商门户项目列表页（含状态Tab），路径：`frontend/src/pages/portal/PortalProjectListPage.tsx`
  - **接口范围**：
  - 本任务调用：`GET /api/v1/portal/projects`

- [ ] T128 [US9.1] 创建报名抽屉（640px：联系人姓名、联系电话、邮箱、授权委托书），路径：`frontend/src/pages/portal/PortalEnrollDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/portal/projects/{id}/enroll`

- [ ] T129 [US9.2] 创建技术方案抽屉（900px：企业信息、技术方案表单、资质文件），路径：`frontend/src/pages/portal/PortalTechProposalDrawer.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/portal/tech-proposals`、`PUT .../tech-proposals/{id}`、`POST .../tech-proposals/{id}/submit`

- [ ] T130 [US9.3] 创建报价页面（全屏：报价表格、自动计算字段、预览确认弹窗、Excel导入），路径：`frontend/src/pages/portal/PortalQuotePage.tsx`
  - **接口范围**：
  - 本任务调用：`POST /api/v1/portal/quotes`、`POST /api/v1/portal/quotes/import`

**检查点**：供应商从登录到报价提交的完整参与流程可用

---

## 阶段十一：P2/P3 模块（仅规划，implement 阶段暂缓执行）

> 以下所有模块的任务均标记 ⏸ P2 暂缓，implement 阶段执行 P1 时跳过。
> P2 开发阶段开始时，直接取消 ⏸ 标记执行对应任务。

---

### 11-A 模块五：保证金管理（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/deposit/deposits | 保证金列表（项目/供应商维度） | US-5.1 | ❌ P2跳过 |
| GET | /api/v1/deposit/deposits/{id} | 保证金详情 | US-5.1 | ❌ P2跳过 |
| POST | /api/v1/deposit/deposits/{id}/pay | 新增缴纳记录 | US-5.1 | ❌ P2跳过 |
| POST | /api/v1/deposit/deposits/{id}/refund | 发起退款申请 | US-5.2 | ❌ P2跳过 |
| POST | /api/v1/deposit/deposits/{id}/refund/approve | 审核退款 | US-5.2 | ❌ P2跳过 |
| POST | /api/v1/deposit/deposits/{id}/confiscate | 没收保证金 | US-5.3 | ❌ P2跳过 |
| GET | /api/v1/deposit/deposits/statistics | 保证金统计 | US-5.1 | ❌ P2跳过 |
| GET | /api/v1/deposit/deposits/export | 导出Excel/CSV | US-5.1 | ❌ P2跳过 |

#### 任务列表

- [ ] T141 [P] [US5.1] 创建 DepositEntity（表名：srm_dep_deposit，字段：project_id、supplier_id、type TD/QZ、required_amount、paid_amount、status、project_stage），路径：`backend/src/main/java/com/srm/deposit/entity/DepositEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T142 [P] [US5.1] 创建 DepositPaymentEntity（表名：srm_dep_payment，字段：deposit_id、amount、pay_date、pay_method、bank_ref、voucher_file_id）和 DepositRefundEntity（表名：srm_dep_refund，字段：deposit_id、reason、status、payment_ids），路径：`backend/src/main/java/com/srm/deposit/entity/DepositPaymentEntity.java` 和 `DepositRefundEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T143 [US5.1] 创建 Flyway 迁移脚本 `V11__deposit_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T144 [US5.1] 创建 DepositService（缴纳记录管理含进度计算、退款申请→Camunda、没收、批量退款），路径：`backend/src/main/java/com/srm/deposit/service/DepositService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T145 [US5.1] 创建 DepositResource，路径：`backend/src/main/java/com/srm/deposit/resource/DepositResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET/POST /api/v1/deposit/deposits`、`GET .../deposits/{id}`、`POST .../deposits/{id}/pay`、`POST .../deposits/{id}/refund`、`POST .../deposits/{id}/refund/approve`、`POST .../deposits/{id}/confiscate`、`GET .../deposits/statistics`、`GET .../deposits/export`
  - 本任务跳过：无

- [ ] T146 [P] [US5.1] 创建保证金列表页（项目/供应商维度Tab，含进度条可视化），路径：`frontend/src/pages/deposit/DepositListPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/deposit/deposits`、`GET .../deposits/statistics`

---

### 11-B 模块七：预算管理（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/budget/budgets | 预算列表 | US-7.1 | ❌ P2跳过 |
| POST | /api/v1/budget/budgets | 新增预算 | US-7.1 | ❌ P2跳过 |
| PUT | /api/v1/budget/budgets/{id} | 更新预算 | US-7.1 | ❌ P2跳过 |
| POST | /api/v1/budget/budgets/{id}/submit | 提交预算审批 | US-7.1 | ❌ P2跳过 |
| POST | /api/v1/budget/budgets/{id}/approve | 审批预算 | US-7.2 | ❌ P2跳过 |
| GET | /api/v1/budget/execution | 预算执行监控 | US-7.3 | ❌ P2跳过 |
| GET | /api/v1/budget/statistics | 预算统计分析 | US-7.4 | ❌ P3跳过 |
| POST | /api/v1/budget/budgets/{id}/adjust | 预算调整 | US-7.1 | ❌ P2跳过 |
| POST | /api/v1/budget/over-budget-requests | 超预算申请 | US-7.5 | ❌ P2跳过 |
| POST | /api/v1/budget/over-budget-requests/{id}/approve | 超预算审批 | US-7.5 | ❌ P2跳过 |

#### 任务列表

- [ ] T147 [P] [US7.1] 创建 BudgetEntity（表名：srm_bgt_budget，字段：project_name、project_number、department、funding_source、subject_json、description、status）和 BudgetAdjustEntity（表名：srm_bgt_adjust），路径：`backend/src/main/java/com/srm/budget/entity/BudgetEntity.java` 和 `BudgetAdjustEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T148 [P] [US7.5] 创建 OverBudgetRequestEntity（表名：srm_bgt_over_budget_request），路径：`backend/src/main/java/com/srm/budget/entity/OverBudgetRequestEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T149 [US7.1] 创建 Flyway 迁移脚本 `V12__budget_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T150 [US7.1] 创建 BudgetService（CRUD、提交审批→Camunda、处理回调、预算调整、执行率计算 85%/100% 阈值）和 OverBudgetService（申请→Camunda、解除拦截），路径：`backend/src/main/java/com/srm/budget/service/BudgetService.java` 和 `OverBudgetService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T151 [US7.1] 创建 BudgetResource，路径：`backend/src/main/java/com/srm/budget/resource/BudgetResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET/POST /api/v1/budget/budgets`、`PUT .../budgets/{id}`、`POST .../budgets/{id}/submit`、`POST .../budgets/{id}/approve`、`GET .../execution`、`GET .../statistics`、`POST .../budgets/{id}/adjust`、`POST .../over-budget-requests`、`POST .../over-budget-requests/{id}/approve`
  - 本任务跳过：无

- [ ] T152 [P] [US7.1] 创建预算列表页（含草稿/审批中/已通过Tab），路径：`frontend/src/pages/budget/BudgetListPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/budget/budgets`

- [ ] T153 [P] [US7.3] 创建预算执行监控页（含5个统计卡片、执行率颜色标记），路径：`frontend/src/pages/budget/BudgetExecutionPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/budget/execution`

- [ ] T154 [P] [US7.4] 创建预算统计页（3张图表），路径：`frontend/src/pages/budget/BudgetStatisticsPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/budget/statistics`

---

### 11-C 模块八：模板管理（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/template/tech-templates | 技术方案模板列表 | US-8.1 | ❌ P2跳过 |
| POST | /api/v1/template/tech-templates | 创建技术方案模板 | US-8.1 | ❌ P2跳过 |
| PUT | /api/v1/template/tech-templates/{id} | 编辑模板 | US-8.1 | ❌ P2跳过 |
| POST | /api/v1/template/tech-templates/{id}/publish | 发布模板 | US-8.1 | ❌ P2跳过 |
| GET | /api/v1/template/quote-templates | 报价模板列表 | US-8.2 | ❌ P2跳过 |
| POST | /api/v1/template/quote-templates | 创建报价模板 | US-8.2 | ❌ P2跳过 |
| PUT | /api/v1/template/quote-templates/{id} | 编辑报价模板 | US-8.2 | ❌ P2跳过 |
| GET | /api/v1/template/score-templates | 评分模板列表 | US-8.3 | ❌ P2跳过 |
| POST | /api/v1/template/score-templates | 创建评分模板 | US-8.3 | ❌ P2跳过 |
| PUT | /api/v1/template/score-templates/{id} | 编辑评分模板 | US-8.3 | ❌ P2跳过 |
| GET | /api/v1/template/templates/{id}/versions | 模板版本历史 | US-8.4 | ❌ P2跳过 |
| POST | /api/v1/template/templates/{id}/rollback | 基于历史版本创建草稿 | US-8.4 | ❌ P2跳过 |

#### 任务列表

- [ ] T155 [P] [US8.1] 创建 TechTemplateEntity（表名：srm_tpl_tech，字段：name、applicable_category、description、instructions、schema_json、version、status），路径：`backend/src/main/java/com/srm/template/entity/TechTemplateEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T156 [P] [US8.2] 创建 QuoteTemplateEntity（表名：srm_tpl_quote，字段：name、columns_json、version、status）和 ScoreTemplateEntity（表名：srm_tpl_score，字段：name、total_score、multi_expert_method、dimensions_json、version、status），路径：`backend/src/main/java/com/srm/template/entity/QuoteTemplateEntity.java` 和 `ScoreTemplateEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T157 [US8.1] 创建 Flyway 迁移脚本 `V13__template_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T158 [US8.1] 创建 TemplateService（CRUD、发布含版本号、版本历史、回滚创建新草稿），路径：`backend/src/main/java/com/srm/template/service/TemplateService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T159 [US8.1] 创建 TemplateResource，实现所有模板接口，路径：`backend/src/main/java/com/srm/template/resource/TemplateResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET/POST/PUT /api/v1/template/tech-templates`、`POST .../tech-templates/{id}/publish`、`GET/POST/PUT /api/v1/template/quote-templates`、`GET/POST/PUT /api/v1/template/score-templates`、`GET .../templates/{id}/versions`、`POST .../templates/{id}/rollback`
  - 本任务跳过：无

- [ ] T160 [P] [US8.1] 创建技术方案模板设计器（拖拽式表单构建器，12种组件类型，字段属性面板，预览），路径：`frontend/src/pages/template/TechTemplateDesigner.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`POST /api/v1/template/tech-templates`、`PUT .../tech-templates/{id}`、`POST .../tech-templates/{id}/publish`

- [ ] T161 [P] [US8.2] 创建报价模板设计器（列编辑器：文本/数字/百分比/公式，按角色显示单元格），路径：`frontend/src/pages/template/QuoteTemplateDesigner.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`POST /api/v1/template/quote-templates`、`PUT .../quote-templates/{id}`

- [ ] T162 [P] [US8.3] 创建评分模板设计器（维度/指标树形结构，权重编辑器，多专家结果处理方式选择），路径：`frontend/src/pages/template/ScoreTemplateDesigner.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`POST /api/v1/template/score-templates`、`PUT .../score-templates/{id}`

---

### 11-D 模块十一：合同管理（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/contract/contracts | 合同列表 | US-11.1 | ❌ P2跳过 |
| POST | /api/v1/contract/contracts | 创建合同（4步向导） | US-11.2 | ❌ P2跳过 |
| GET | /api/v1/contract/contracts/{id} | 合同详情 | US-11.1 | ❌ P2跳过 |
| PUT | /api/v1/contract/contracts/{id} | 更新合同 | US-11.2 | ❌ P2跳过 |
| POST | /api/v1/contract/contracts/{id}/submit | 提交合同审批 | US-11.2 | ❌ P2跳过 |
| POST | /api/v1/contract/contracts/{id}/change | 发起合同变更 | US-11.3 | ❌ P2跳过 |
| GET | /api/v1/contract/contracts/statistics | 合同统计 | US-11.1 | ❌ P2跳过 |
| GET | /api/v1/contract/clauses | 条款库列表 | US-11.4 | ❌ P2跳过 |
| POST | /api/v1/contract/clauses | 新增条款 | US-11.4 | ❌ P2跳过 |
| GET | /api/v1/contract/templates | 合同模板列表 | US-11.5 | ❌ P2跳过 |
| POST | /api/v1/contract/templates | 创建合同模板 | US-11.5 | ❌ P2跳过 |
| GET | /api/v1/contract/keywords | 关键字列表 | US-11.6 | ❌ P3跳过 |
| POST | /api/v1/contract/keywords | 新增关键字 | US-11.6 | ❌ P3跳过 |
| POST | /api/v1/contract/contracts/import | 文件导入合同 | US-11.2 | ❌ P2跳过 |
| GET | /api/v1/contract/contracts/{id}/supplements | 补充协议列表 | US-11.2 | ❌ P2跳过 |
| POST | /api/v1/contract/contracts/{id}/supplements | 新增补充协议 | US-11.2 | ❌ P2跳过 |
| GET | /api/v1/contract/contracts/{id}/changes | 变更记录列表 | US-11.3 | ❌ P2跳过 |
| POST | /api/v1/contract/contracts/{id}/void | 合同作废申请 | US-11.3 | ❌ P2跳过 |

#### 任务列表

- [ ] T163 [P] [US11.1] 创建 ContractEntity（表名：srm_ctr_contract，字段：name、contract_number、type、amount、start_date、end_date、summary、party_a_name、party_a_credit_code、party_b_name、party_b_contact、related_project_id、content_json、status），路径：`backend/src/main/java/com/srm/contract/entity/ContractEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T164 [P] [US11.2] 创建 SupplementEntity（表名：srm_ctr_supplement，字段：contract_id、supplement_number、name、type、modified_clauses_json、additional_amount、status）和 ContractChangeEntity（表名：srm_ctr_change，字段：contract_id、change_type、title、reason、detail、impact、change_amount、expected_effective_date、status、ip_address），路径：`backend/src/main/java/com/srm/contract/entity/SupplementEntity.java` 和 `ContractChangeEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T165 [P] [US11.4] 创建 ClauseEntity（表名：srm_ctr_clause，字段：title、category、clause_type FIXED/KEYWORD、content_json、version、status）和 ContractKeywordEntity（表名：srm_ctr_keyword，字段：name、code、data_type、default_value、is_required、is_editable、is_enabled、reference_count），路径：`backend/src/main/java/com/srm/contract/entity/ClauseEntity.java` 和 `ContractKeywordEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T166 [P] [US11.5] 创建 ContractTemplateEntity（表名：srm_ctr_template，字段：name、type、applicable_department、applicable_scenario、amount_range_min、amount_range_max、content_json、version、status），路径：`backend/src/main/java/com/srm/contract/entity/ContractTemplateEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T167 [US11.1] 创建 Flyway 迁移脚本 `V15__contract_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T168 [US11.1] 创建 ContractService（CRUD、提交审批→Camunda 含100万大额自动5级审批、处理回调、发起变更含4级审批、文件导入、获取统计）和 ClauseService（CRUD、提交审核、处理审核回调、批量导入），路径：`backend/src/main/java/com/srm/contract/service/ContractService.java` 和 `ClauseService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T169 [US11.1] 创建 ContractResource 和 ClauseResource，路径：`backend/src/main/java/com/srm/contract/resource/ContractResource.java` 和 `ClauseResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET/POST /api/v1/contract/contracts`、`GET/PUT .../contracts/{id}`、`POST .../contracts/{id}/submit`、`POST .../contracts/{id}/change`、`GET .../contracts/statistics`、`POST .../contracts/import`、`GET/POST .../contracts/{id}/supplements`、`GET .../contracts/{id}/changes`、`POST .../contracts/{id}/void`、`GET/POST /api/v1/contract/clauses`、`GET/POST /api/v1/contract/templates`、`GET/POST /api/v1/contract/keywords`
  - 本任务跳过：无

- [ ] T170 [P] [US11.1] 创建合同列表页（含统计卡片、状态Tab、可展开补充协议子表），路径：`frontend/src/pages/contract/ContractListPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/contract/contracts`、`GET .../contracts/statistics`

- [ ] T171 [US11.2] 创建合同创建向导（4步：模板/导入→基本信息→内容编辑器→提交审批，含块编辑器），路径：`frontend/src/pages/contract/ContractWizard.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`POST /api/v1/contract/contracts`、`PUT .../contracts/{id}`、`POST .../contracts/{id}/submit`

- [ ] T172 [P] [US11.1] 创建合同详情抽屉（5个Tab：预览/基本信息/补充协议/变更记录/流程进展），路径：`frontend/src/pages/contract/ContractDetailDrawer.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET .../contracts/{id}`、`GET .../contracts/{id}/supplements`、`GET .../contracts/{id}/changes`

- [ ] T173 [P] [US11.4] 创建条款库列表页（含统计、版本对比、批量导入），路径：`frontend/src/pages/contract/ClauseListPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/contract/clauses`、`POST .../clauses`

- [ ] T174 [P] [US11.5] 创建合同模板列表页（含三栏编辑器：目录+画布+条款库），路径：`frontend/src/pages/contract/ContractTemplateListPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/contract/templates`、`POST .../templates`

---

### 11-E 模块十二：电子签章（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| POST | /api/v1/esign/contracts/{id}/initiate | 发起签署 | US-12.1 | ❌ P2跳过 |
| POST | /api/v1/esign/contracts/{id}/sign | 在线签署 | US-12.2 | ❌ P2跳过 |
| POST | /api/v1/esign/contracts/{id}/remind | 催签提醒 | US-12.1 | ❌ P2跳过 |
| POST | /api/v1/esign/contracts/{id}/revoke | 撤回签署 | US-12.1 | ❌ P2跳过 |
| GET | /api/v1/esign/contracts/{id}/status | 签署状态查询 | US-12.1 | ❌ P2跳过 |
| GET | /api/v1/esign/contracts/{id}/evidence | 签署存证查询 | US-12.2 | ❌ P2跳过 |
| POST | /api/v1/esign/callback | e签宝签署回调 | — | ❌ P2跳过（外部回调） |
| POST | /api/v1/esign/contracts/{id}/terminate | 终止签署流程 | US-12.5 | ❌ P2跳过 |

#### 任务列表

- [ ] T175 [P] [US12.1] 创建 EsignRecordEntity（表名：srm_esign_record，字段：contract_id、signer_name、signer_type PARTY_A/PARTY_B、sign_method、ca_cert_sn、blockchain_hash、status、initiated_at、signed_at、deadline），路径：`backend/src/main/java/com/srm/esign/entity/EsignRecordEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T176 [US12.1] 创建 Flyway 迁移脚本 `V16__esign_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T177 [US12.1] 创建 EsignService（发起签署→调用e签宝API、签署、催签、撤回、终止、处理回调含幂等状态校验、查询状态、查询存证），路径：`backend/src/main/java/com/srm/esign/service/EsignService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T178 [US12.1] 创建 EsignResource，含所有签章接口和回调接收端，路径：`backend/src/main/java/com/srm/esign/resource/EsignResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`POST .../contracts/{id}/initiate`、`POST .../contracts/{id}/sign`、`POST .../contracts/{id}/remind`、`POST .../contracts/{id}/revoke`、`GET .../contracts/{id}/status`、`GET .../contracts/{id}/evidence`、`POST /api/v1/esign/callback`（外部回调接收端）、`POST .../contracts/{id}/terminate`
  - 本任务跳过：无

- [ ] T179 [P] [US12.1] 创建签署状态页（签署进度追踪、签署方列表、操作按钮：催签/撤回/终止），路径：`frontend/src/pages/esign/EsignStatusPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET .../contracts/{id}/status`、`POST .../contracts/{id}/initiate`、`POST .../contracts/{id}/remind`

---

### 11-F 模块十三：档案管理（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/archive/archives | 档案列表（含OCR全文检索） | US-13.1 | ❌ P2跳过 |
| GET | /api/v1/archive/archives/{id} | 档案详情 | US-13.1 | ❌ P2跳过 |
| GET | /api/v1/archive/rules | 归档规则列表 | US-13.2 | ❌ P2跳过 |
| POST | /api/v1/archive/rules | 创建归档规则 | US-13.2 | ❌ P2跳过 |
| PUT | /api/v1/archive/rules/{id} | 更新归档规则 | US-13.2 | ❌ P2跳过 |
| GET | /api/v1/archive/statistics | 档案统计 | US-13.3 | ❌ P2跳过 |
| POST | /api/v1/archive/archives/{id}/borrow | 申请借阅 | US-13.4 | ❌ P2跳过 |
| POST | /api/v1/archive/archives/{id}/return | 归还借阅 | US-13.4 | ❌ P2跳过 |
| GET | /api/v1/archive/borrows | 借阅记录列表 | US-13.4 | ❌ P2跳过 |
| GET | /api/v1/archive/archives/{id}/access-log | 访问日志 | US-13.1 | ❌ P2跳过 |

#### 任务列表

- [ ] T180 [P] [US13.1] 创建 ArchiveEntity（表名：srm_arc_archive，字段：title、archive_number、category、confidentiality_level、retention_years、sha256_hash、file_id、physical_location、status、source_type、source_id）和 ArchiveAccessLogEntity（表名：srm_arc_access_log，字段：archive_id、operator_id、access_type、ip_address、result），路径：`backend/src/main/java/com/srm/archive/entity/ArchiveEntity.java` 和 `ArchiveAccessLogEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T181 [P] [US13.2] 创建 ArchiveRuleEntity（表名：srm_arc_rule，字段：category_name、biz_types_json、default_retention_years、default_confidentiality、force_archive）和 BorrowRecordEntity（表名：srm_arc_borrow，字段：archive_id、borrower_id、reason、expected_return_date、actual_return_date、status），路径：`backend/src/main/java/com/srm/archive/entity/ArchiveRuleEntity.java` 和 `BorrowRecordEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T182 [US13.1] 创建 Flyway 迁移脚本 `V17__archive_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T183 [US13.1] 创建 ArchiveService（列表含OCR全文检索、详情含SHA256重新校验、访问日志记录、归档统计）和 BorrowService（申请→审核→借阅中、归还、排队、超期催还），路径：`backend/src/main/java/com/srm/archive/service/ArchiveService.java` 和 `BorrowService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T184 [US13.1] 创建 ArchiveResource，路径：`backend/src/main/java/com/srm/archive/resource/ArchiveResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET /api/v1/archive/archives`、`GET .../archives/{id}`、`GET/POST/PUT /api/v1/archive/rules`、`GET .../statistics`、`POST .../archives/{id}/borrow`、`POST .../archives/{id}/return`、`GET .../borrows`、`GET .../archives/{id}/access-log`
  - 本任务跳过：无

- [ ] T185 [P] [US13.1] 创建档案列表页（含5个统计卡片、OCR全文检索框、状态Tab），路径：`frontend/src/pages/archive/ArchiveListPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/archive/archives`

- [ ] T186 [P] [US13.2] 创建归档规则管理页，路径：`frontend/src/pages/archive/ArchiveRulePage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET/POST/PUT /api/v1/archive/rules`

---

### 11-G 模块十四：供应商绩效管理（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/performance/dashboard | 绩效总览看板 | US-14.1 | ❌ P2跳过 |
| GET | /api/v1/performance/scores | 绩效评分列表 | US-14.3 | ❌ P2跳过 |
| POST | /api/v1/performance/scores | 提交绩效评分 | US-14.3 | ❌ P2跳过 |
| GET | /api/v1/performance/events | 绩效事件列表 | US-14.2 | ❌ P2跳过 |
| POST | /api/v1/performance/events | 创建绩效事件 | US-14.2 | ❌ P2跳过 |
| POST | /api/v1/performance/events/{id}/approve | 审核绩效事件 | US-14.2 | ❌ P2跳过 |
| GET | /api/v1/performance/models | 绩效模型列表 | US-14.4 | ❌ P2跳过 |
| POST | /api/v1/performance/models | 创建绩效模型 | US-14.4 | ❌ P2跳过 |
| PUT | /api/v1/performance/models/{id} | 更新绩效模型 | US-14.4 | ❌ P2跳过 |
| GET | /api/v1/performance/models/{id}/versions | 模型版本历史 | US-14.4 | ❌ P2跳过 |

#### 任务列表

- [ ] T187 [P] [US14.1] 创建 PerformanceScoreEntity（表名：srm_perf_score，字段：supplier_id、period_year、period_month、delivery_score、quality_score、cost_score、service_score、event_deduction、composite_score、grade、model_id）和 PerformanceEventEntity（表名：srm_perf_event，字段：supplier_id、event_type、title、occurred_date、description、affects_performance、deduction_score、period_year、period_month、status），路径：`backend/src/main/java/com/srm/performance/entity/PerformanceScoreEntity.java` 和 `PerformanceEventEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T188 [P] [US14.4] 创建 PerformanceModelEntity（表名：srm_perf_model，字段：name、version、is_default、dimensions_json、grade_thresholds_json、status），路径：`backend/src/main/java/com/srm/performance/entity/PerformanceModelEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T189 [US14.1] 创建 Flyway 迁移脚本 `V18__performance_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T190 [US14.1] 创建 PerformanceService（月度评分执行含4维度加权+事件扣分、D/F级联动拦截、获取看板统计）和 PerformanceEventService（CRUD、审核含锁定、关联绩效指标自动统计），路径：`backend/src/main/java/com/srm/performance/service/PerformanceService.java` 和 `PerformanceEventService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T191 [US14.1] 创建 PerformanceResource，路径：`backend/src/main/java/com/srm/performance/resource/PerformanceResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET /api/v1/performance/dashboard`、`GET/POST .../scores`、`GET/POST .../events`、`POST .../events/{id}/approve`、`GET/POST .../models`、`PUT .../models/{id}`、`GET .../models/{id}/versions`
  - 本任务跳过：无

- [ ] T192 [P] [US14.1] 创建绩效总览看板页（含5个统计卡片、近6月趋势折线图、等级分布饼图），路径：`frontend/src/pages/performance/PerformanceDashboard.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/performance/dashboard`

- [ ] T193 [P] [US14.3] 创建绩效评分执行页（4维度打分、实时计算器、事件扣分联动），路径：`frontend/src/pages/performance/PerformanceScoringPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET .../scores`、`POST .../scores`

- [ ] T194 [P] [US14.2] 创建绩效事件管理页（含统计、事件类型色标），路径：`frontend/src/pages/performance/PerformanceEventPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET/POST /api/v1/performance/events`、`POST .../events/{id}/approve`

---

### 11-H 模块十五：风险与治理中心（P2）

#### API 接口清单

| 方法 | 路径 | 说明 | 关联US | 实现 |
|------|------|------|--------|------|
| GET | /api/v1/risk/contract-risks | 合同风险监控列表 | US-15.1 | ❌ P2跳过 |
| GET | /api/v1/risk/supplier-risks | 供应商风险列表 | US-15.2 | ❌ P2跳过 |
| POST | /api/v1/risk/supplier-risks/{id}/assess | 执行风险评估 | US-15.4 | ❌ P2跳过 |
| GET | /api/v1/risk/alerts | 预警中心列表 | US-15.2 | ❌ P2跳过 |
| PATCH | /api/v1/risk/alerts/{id}/status | 更新预警状态 | US-15.2 | ❌ P2跳过 |
| GET | /api/v1/risk/rectifications | 整改任务列表 | US-15.3 | ❌ P2跳过 |
| POST | /api/v1/risk/rectifications | 创建整改任务 | US-15.3 | ❌ P2跳过 |
| POST | /api/v1/risk/rectifications/{id}/follow-up | 添加跟进记录 | US-15.3 | ❌ P2跳过 |
| GET | /api/v1/risk/disputes | 争议谈判列表 | US-15.3 | ❌ P2跳过 |
| POST | /api/v1/risk/disputes | 创建争议记录 | US-15.3 | ❌ P2跳过 |
| POST | /api/v1/risk/disputes/{id}/rounds | 添加谈判轮次 | US-15.3 | ❌ P2跳过 |
| GET | /api/v1/risk/risk-matrix | 风险矩阵数据 | US-15.4 | ❌ P2跳过 |

#### 任务列表

- [ ] T195 [P] [US15.1] 创建 SupplierRiskEntity（表名：srm_risk_supplier，字段：supplier_id、financial_health、supply_stability、quality_compliance、geopolitical_risk、concentration_risk、composite_score、risk_level、assessed_by、assessed_at）和 RiskAlertEntity（表名：srm_risk_alert，字段：supplier_id、alert_type、priority P1-P4、title、status、source），路径：`backend/src/main/java/com/srm/risk/entity/SupplierRiskEntity.java` 和 `RiskAlertEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T196 [P] [US15.3] 创建 RectificationEntity（表名：srm_risk_rectification，字段：supplier_id、title、status、due_date、follow_up_json）和 RectificationFollowUpEntity（表名：srm_risk_rectification_followup），路径：`backend/src/main/java/com/srm/risk/entity/RectificationEntity.java` 和 `RectificationFollowUpEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T197 [P] [US15.2] 创建 DisputeEntity（表名：srm_risk_dispute，字段：supplier_id、title、status、current_round）和 DisputeRoundEntity（表名：srm_risk_dispute_round，字段：dispute_id、round_number、our_position、their_position、result、approval_level），路径：`backend/src/main/java/com/srm/risk/entity/DisputeEntity.java` 和 `DisputeRoundEntity.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Entity 层

- [ ] T198 [US15.1] 创建 Flyway 迁移脚本 `V19__risk_tables.sql`，路径：`backend/src/main/resources/db/migration/` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅数据库迁移

- [ ] T199 [US15.1] 创建 RiskService（合同风险监控含到期提醒、供应商风险评估含5维度评分、预警管理 P1-P4、整改任务追踪含跟进记录、争议管理含轮次、获取5×5风险矩阵），路径：`backend/src/main/java/com/srm/risk/service/RiskService.java` ⏸ P2 暂缓
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T200 [US15.1] 创建 RiskResource，路径：`backend/src/main/java/com/srm/risk/resource/RiskResource.java` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务实现：`GET /api/v1/risk/contract-risks`、`GET .../supplier-risks`、`POST .../supplier-risks/{id}/assess`、`GET .../alerts`、`PATCH .../alerts/{id}/status`、`GET/POST .../rectifications`、`POST .../rectifications/{id}/follow-up`、`GET/POST .../disputes`、`POST .../disputes/{id}/rounds`、`GET .../risk-matrix`
  - 本任务跳过：无

- [ ] T201 [P] [US15.1] 创建合同风险监控页（风险列表、到期倒计时、提醒配置），路径：`frontend/src/pages/risk/ContractRiskPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET /api/v1/risk/contract-risks`

- [ ] T202 [P] [US15.2] 创建供应商风险页和预警中心页（P1-P4优先级Tab），路径：`frontend/src/pages/risk/SupplierRiskPage.tsx` 和 `AlertCenterPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET .../supplier-risks`、`GET .../alerts`、`PATCH .../alerts/{id}/status`

- [ ] T203 [P] [US15.2] 创建整改任务页和争议谈判页（含跟进时间线、轮次管理），路径：`frontend/src/pages/risk/RectificationPage.tsx` 和 `DisputePage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET/POST .../rectifications`、`POST .../rectifications/{id}/follow-up`、`GET/POST .../disputes`、`POST .../disputes/{id}/rounds`

- [ ] T204 [P] [US15.4] 创建风险矩阵页（5×5矩阵可视化、评估执行表单），路径：`frontend/src/pages/risk/RiskMatrixPage.tsx` ⏸ P2 暂缓
  - **接口范围**：
  - 本任务调用：`GET .../risk-matrix`、`POST .../supplier-risks/{id}/assess`

---

## 阶段十二：收尾与横切关注点

**目标**：多个用户故事的通用改进

- [ ] T131 [P] 实现数据脱敏（手机号中间4位替换为****，邮箱@前部分隐藏），路径：`backend/src/main/java/com/srm/common/security/DataMaskingUtil.java`
  - **接口范围**：本任务不涉及接口，仅实现工具类

- [ ] T132 [P] 创建403无权限页面和未授权跳转，路径：`frontend/src/pages/error/ForbiddenPage.tsx`
  - **接口范围**：本任务不涉及接口，仅前端页面

- [ ] T133 [P] 创建危险操作通用确认组件，路径：`frontend/src/components/common/ConfirmAction.tsx`
  - **接口范围**：本任务不涉及接口，仅前端组件

- [ ] T134 [P] 创建消息通知铃铛组件（未读数徽章、下拉列表、标记已读），路径：`frontend/src/components/common/NotificationBell.tsx`
  - **接口范围**：本任务不涉及接口，仅前端组件

- [ ] T135 [P] 创建空态和加载骨架屏组件，路径：`frontend/src/components/common/EmptyState.tsx` 和 `LoadingSkeleton.tsx`
  - **接口范围**：本任务不涉及接口，仅前端组件

- [ ] T136 [P] 创建 Toast 通知工具（3秒自动消失），路径：`frontend/src/utils/toast.ts`
  - **接口范围**：本任务不涉及接口，仅前端工具

- [ ] T137 [P] 创建清理定时任务（孤立文件72小时、审计日志3个月），路径：`backend/src/main/java/com/srm/common/scheduler/CleanupScheduler.java`
  - **接口范围**：本任务不涉及接口，仅实现定时任务

- [ ] T138 [P] 创建导出服务，支持按筛选条件导出全量 Excel（不受分页限制），路径：`backend/src/main/java/com/srm/common/service/ExportService.java`
  - **接口范围**：本任务不涉及接口，仅实现 Service 层

- [ ] T139 验证所有 API 接口均返回统一 ApiResponse 格式和正确 HTTP 状态码
  - **接口范围**：本任务验证所有已实现接口的响应格式一致性

- [ ] T140 运行 quickstart.md 验收测试：Docker Compose 启动→默认账号→创建申请→保存草稿
  - **接口范围**：本任务为端到端冒烟测试

---

## 依赖关系与执行顺序

### 阶段依赖

```text
阶段一（初始化）
  → 阶段二（基础设施）⚠️ 阻塞所有后续阶段
    → 阶段三（认证）🎯 最小可用版本
      ├── 阶段四（供应商管理）─────┐
      └── 阶段五（采购需求）────────┤
                                   ├── 阶段六（寻源项目）
                                   │     ├── 阶段七（技术方案）──┐
                                   │     └── 阶段八（报价竞价）──┤
                                   │                             ├── 阶段九（招投标）
                                   │                             └── 阶段十（供应商门户）
                                   └──────────────────────────── 阶段十二（收尾）
```

### 并行机会

- 阶段四 + 阶段五 可同时进行（认证完成后）
- 阶段七 + 阶段八 可同时进行（寻源项目完成后）
- 阶段九 + 阶段十 可部分并行（阶段十不依赖阶段九）
- 每个阶段内标记 [P] 的 Entity 层/前端任务可并行执行

---

## 实施策略

### 优先最小可用版本（阶段一至三）

1. 阶段一：初始化 → 阶段二：基础设施 → 阶段三：认证
2. **停止验证**：确认登录和 JWT 认证正常工作
3. 可部署演示

### 增量交付

1. 认证 ✅ → 2. 供应商 ✅ → 3. 采购需求 ✅ → 4. 寻源项目 ✅
5. 技术方案 + 报价竞价（并行）✅ → 6. 招投标 ✅ → 7. 供应商门户 ✅ → 8. 收尾 ✅

---

## 任务汇总

| 阶段 | 模块 | 任务数 | 可并行 | P1接口数 |
|------|------|--------|--------|---------|
| 1 | 初始化 | 5 | 4 | 0 |
| 2 | 基础设施 | 22 | 15 | 7 |
| 3 | 认证 | 14 | 5 | 5 |
| 4 | 供应商管理 | 12 | 4 | 8 |
| 5 | 采购需求管理 | 12 | 4 | 12 |
| 6 | 寻源项目管理 | 13 | 5 | 11 |
| 7 | 技术方案管理 | 14 | 5 | 11 |
| 8 | 报价与竞价管理 | 16 | 5 | 9+1WS |
| 9 | 招投标管理 | 14 | 5 | 11 |
| 10 | 供应商门户 | 8 | 1 | 8 |
| 11-A | 保证金管理 ⏸ | 6 | 2 | 8（已延后） |
| 11-B | 预算管理 ⏸ | 8 | 4 | 10（已延后） |
| 11-C | 模板管理 ⏸ | 8 | 4 | 12（已延后） |
| 11-D | 合同管理 ⏸ | 12 | 5 | 18（已延后） |
| 11-E | 电子签章 ⏸ | 5 | 1 | 8（已延后） |
| 11-F | 档案管理 ⏸ | 7 | 3 | 10（已延后） |
| 11-G | 供应商绩效管理 ⏸ | 8 | 4 | 10（已延后） |
| 11-H | 风险与治理中心 ⏸ | 10 | 5 | 12（已延后） |
| 12 | 收尾 | 10 | 8 | 0 |
| **合计** | | **204** | **89** | **P1：82+1WS / P2：88（已延后）** |
