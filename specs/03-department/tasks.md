# 部门管理 - 任务清单

> 状态说明：`[ ]` 未完成，`[✅]` 已完成

- [ ] 1. 部门表与实体
  - 创建 `DeptEntity`（含 ancestors）与迁移脚本，为 sys_user.dept_id 建立索引
  - 涉及文件：`server/src/modules/dept/dept.entity.ts`
  - 对应需求：FR-1、FR-2
- [ ] 2. 部门查询接口
  - [ ] 2.1 实现 `GET /api/depts`（树构建、按名称/状态筛选并保留祖先）
  - [ ] 2.2 实现 `GET /api/depts/options` 与 `GET /api/depts/:id`
  - 对应需求：FR-1
- [ ] 3. 新增与编辑
  - [ ] 3.1 实现 `POST /api/depts`：同级名称唯一校验，生成 ancestors
  - [ ] 3.2 实现 `PUT /api/depts/:id`：父级循环校验、子树 ancestors 更新、停用校验
  - 对应需求：FR-2
- [ ] 4. 删除
  - 实现 `DELETE /api/depts/:id`：子部门与用户校验
  - 对应需求：FR-3
- [ ] 5. 子树查询能力
  - 实现并导出 `DeptService.getSubtreeIds`
  - 对应需求：FR-4
- [ ] 6. 前端部门管理页
  - [ ] 6.1 实现树形表格与筛选
  - [ ] 6.2 实现新增/编辑弹窗（父级选择禁用自身及下级）
  - [ ] 6.3 抽出 `DeptTreeSelect` 组件
  - 涉及文件：`web/src/views/system/dept/`、`web/src/components/DeptTreeSelect.vue`、`web/src/api/dept.ts`
  - 对应需求：FR-1、FR-2、FR-3
- [ ] 7. 编写测试并通过
  - [ ] 7.1 单测：树构建、循环校验、ancestors 更新
  - [ ] 7.2 e2e：删除校验、增改查流程
  - 对应需求：FR-1 ~ FR-4
