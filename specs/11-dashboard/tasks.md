# 首页仪表盘 - 任务清单

> 状态说明：`[ ]` 未完成，`[✅]` 已完成

- [ ] 1. 统计接口
  - [ ] 1.1 实现 `GET /api/dashboard/stats`
  - [ ] 1.2 实现 `GET /api/dashboard/login-trend`（补齐缺失的日期）
  - [ ] 1.3 实现 `GET /api/dashboard/recent-operations`
  - [ ] 1.4 为三个接口加上 60 秒 Redis 缓存
  - 涉及文件：`server/src/modules/dashboard/`
  - 对应需求：FR-1、FR-2、FR-3
- [ ] 2. 权限与种子数据
  - 新增 `dashboard:view` 权限标识并授予系统管理员角色
  - 对应需求：FR-4
- [ ] 3. 前端首页
  - [ ] 3.1 实现统计卡片（骨架屏、失败重试）
  - [ ] 3.2 实现 `LineChart` 组件（按需引入 ECharts、自适应）与登录趋势图
  - [ ] 3.3 实现最近操作表格与"查看更多"跳转
  - [ ] 3.4 没有权限时只显示欢迎语
  - 涉及文件：`web/src/views/dashboard/`、`web/src/components/LineChart.vue`、`web/src/api/dashboard.ts`
  - 对应需求：FR-1 ~ FR-4
- [ ] 4. 编写测试并通过
  - [ ] 4.1 单测：趋势日期补齐、跨天边界
  - [ ] 4.2 e2e：统计数值正确
  - [ ] 4.3 前端：无权限时不请求统计接口
  - 对应需求：FR-1 ~ FR-4
