# MaxKB 项目结构分析（Agent 快速上手）

## 1. 项目概览
- 技术栈：后端 `Python 3.11 + Django 5`，前端 `Vue 3 + Vite + TypeScript`。
- 核心能力：RAG 知识库、工作流编排、模型管理、工具/MCP 集成、触发器与聊天 API。
- 运行形态：单仓库多模块（后端 `apps/*` + 前端 `ui/*` + 安装脚本 `installer/*`）。

## 2. 顶层目录
- `apps/`：Django 业务代码（核心）。
- `ui/`：Vue 前端工程（管理端 + 聊天端）。
- `installer/`：容器化与启动脚本（Dockerfile、start 脚本、初始化 SQL）。
- `main.py`：统一启动入口（collectstatic、migrate、dev/start）。
- `pyproject.toml`：后端依赖与 Python 构建配置。

## 3. 后端结构（apps）

### 3.1 配置与入口
- `apps/maxkb/const.py`：加载配置来源（`/opt/maxkb/conf` 或环境变量）。
- `apps/maxkb/conf.py`：配置管理（数据库、Redis、时区、路径、运行参数）。
- `apps/maxkb/settings/`：按场景拆分设置：
  - `base/web.py`：主服务（admin/chat API、多 app）。
  - `base/model.py`：本地模型服务（`local_model`）。
- `apps/maxkb/urls/web.py`：主路由聚合，挂载 admin/chat API，并处理前端静态资源路由。
- `apps/maxkb/wsgi/web.py`：Web WSGI 入口，启动后执行事件与模板初始化。
- `apps/maxkb/wsgi/model.py`：模型服务 WSGI 入口。

### 3.2 主要业务模块（按体量）
- `apps/models_provider/`（约 374 文件）：模型接入与管理（多厂商/多协议适配、序列化、SQL、视图）。
- `apps/application/`（约 253 文件）：应用与工作流核心（`flow/step_node` 节点体系、聊天记录、版本、API Key 等）。
- `apps/common/`（约 156 文件）：公共基础层（中间件、鉴权、异常、缓存、工具、数据库通用能力）。
- `apps/knowledge/`（约 80 文件）：知识库与向量检索、任务与文档处理。
- `apps/system_manage/`（约 45 文件）：系统管理、权限与系统配置。
- `apps/trigger/`（约 30 文件）：触发器与调度处理。
- `apps/tools/`（约 25 文件）：工具中心与工具 API。
- `apps/chat/`（约 25 文件）：聊天 API 与模板。
- `apps/users/`：用户与登录。
- `apps/local_model/`：本地模型服务接口。
- `apps/folders/`, `apps/oss/`, `apps/ops/`, `apps/locales/`：目录管理、对象存储、运维/celery、多语言资源。

### 3.3 模块内通用分层（大多数 app）
- `models/`：ORM 模型。
- `serializers/`：DRF 序列化。
- `api/` / `views/`：接口与视图。
- `urls.py`：路由注册。
- `migrations/`：数据库迁移。
- `sql/`：复杂查询 SQL。

## 4. 前端结构（ui）
- 构建与脚本：`ui/package.json`（`dev`、`build`、`chat`、`lint`）。
- 入口文件：`ui/src/main.ts`（管理端）、`ui/src/chat.ts`（聊天端）。
- 路由：`ui/src/router/`（按业务模块拆分）。
- 页面：`ui/src/views/`（约 210 文件）。
- 组件：`ui/src/components/`（约 141 文件）。
- 工作流可视化：`ui/src/workflow/`（约 152 文件）。
- API 调用：`ui/src/api/` + `ui/src/request/`。
- 状态管理：`ui/src/stores/`（Pinia）。

## 5. 启动与运行链路
- 统一入口：`python main.py <action> [services...]`
  - `main.py dev web`：本地 Web 开发（会先执行 `collectstatic` + `migrate`）。
  - `main.py dev celery`：本地任务服务。
  - `main.py start all -d`：按服务模式启动。
- 前端独立开发：
  - `cd ui && npm install && npm run dev`
- 生产容器：`installer/` 下 Dockerfile + start 脚本。

## 6. 配置要点
- 默认配置来自 `config*.yml`，也支持 `MAXKB_` 前缀环境变量。
- 关键依赖：PostgreSQL（含 pgvector）、Redis、Celery。
- 路径前缀可配置：`ADMIN_PATH`、`CHAT_PATH`（影响 URL 与静态资源映射）。

## 7. Agent 改动定位建议
- 改 API：优先定位对应 app 的 `api/ + serializers/ + views/ + urls.py`。
- 改工作流节点：`apps/application/flow/step_node/*`。
- 改模型接入：`apps/models_provider/*`。
- 改知识检索/RAG：`apps/knowledge/*` 与 `apps/application/chat_pipeline/*`。
- 改前端页面：`ui/src/views/*`，公共组件在 `ui/src/components/*`。
- 改权限/系统配置：`apps/system_manage/*` 与 `apps/common/auth/*`。

## 8. 当前结构结论
该仓库是典型的“Django 多应用 + Vue 单页应用”企业级单仓。后端以 `application/models_provider/common` 为核心重心，前端以 `views/workflow/components` 为主要开发面。对于自动化代理协作，建议按“业务 app 边界”进行最小改动，避免跨模块耦合修改。
