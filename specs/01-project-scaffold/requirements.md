# 项目基础框架 - 需求说明

> 来源：PRD.md § 4 非功能需求、§ 5 技术约束（本需求由拆分时补充，PRD 未单列）

## 背景与目标
所有业务模块都依赖同一套前后端工程、数据库/缓存接入、统一的接口规范与页面布局。先把这些公共基础搭好，后续模块只关注业务本身。

## 用户故事
- 作为开发人员，我希望有统一的工程结构和接口规范，以便各模块风格一致、可并行开发。
- 作为使用者，我希望所有操作都有明确的成功/失败反馈，以便知道操作结果。

## 功能需求
### FR-1 后端工程
基于 NestJS + TypeScript 的后端工程，接入 MySQL 8 与 Redis，配置按环境区分。

**验收标准**
- WHEN 执行启动命令 THEN 系统 SHALL 成功连接 MySQL 与 Redis 并监听配置端口
- WHEN 请求 `GET /api/health` THEN 系统 SHALL 返回数据库与 Redis 的连通状态

### FR-2 统一响应与异常
**验收标准**
- WHEN 接口成功 THEN 系统 SHALL 返回 `{ code: 0, message: "ok", data }`
- WHEN 发生业务异常 THEN 系统 SHALL 返回对应业务错误码与可读的中文 message，HTTP 状态码按语义设置（400/401/403/404/409/500）
- WHEN 发生未捕获异常 THEN 系统 SHALL 返回 `code: 500` 且不暴露堆栈信息
- WHEN 请求参数校验失败 THEN 系统 SHALL 返回 400 并指出具体字段

### FR-3 分页与列表约定
**验收标准**
- WHEN 列表接口传入 `page`、`pageSize` THEN 系统 SHALL 返回 `{ list, total, page, pageSize }`，`pageSize` 最大 100

### FR-4 前端工程与布局
基于 Vue 3 + TypeScript + Element Plus + Vite + Pinia + Vue Router 的前端工程。

**验收标准**
- WHEN 访问系统 THEN 系统 SHALL 展示包含侧边栏、顶栏（面包屑、用户菜单）、内容区的基础布局
- WHEN 接口返回非 0 code THEN 前端 SHALL 统一弹出错误提示
- WHEN 执行删除类操作 THEN 前端 SHALL 先弹出二次确认
- WHEN 渲染界面文案 THEN 前端 SHALL 通过 i18n 资源读取（一期仅 zh-CN）

### FR-5 安全基线
**验收标准**
- WHEN 返回 HTML 或接收用户输入 THEN 系统 SHALL 防 XSS（前端不使用 v-html 渲染用户输入，后端对富文本外的输入不做 HTML 渲染）
- WHEN 访问数据库 THEN 系统 SHALL 仅使用参数化查询/ORM，禁止拼接 SQL

## 非功能需求
- 兼容最新两个版本的 Chrome/Edge/Firefox/Safari，最小分辨率 1366×768
- 生产环境强制 HTTPS（部署层实现）

## 范围外
- 部署脚本、CI/CD 流水线
- 业务模块本身

## 依赖
无

## 待确认问题
- ORM 选型：PRD 未指定，设计中假设使用 TypeORM。
- 是否采用 monorepo：设计中假设为单仓库下 `server/` 与 `web/` 两个目录。
