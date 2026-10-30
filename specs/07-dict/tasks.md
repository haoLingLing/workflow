# 数据字典 - 任务清单

> 状态说明：`[ ]` 未完成，`[✅]` 已完成

- [ ] 1. 字典表与种子数据
  - [ ] 1.1 创建 `DictTypeEntity`、`DictDataEntity` 与迁移脚本
  - [ ] 1.2 种子数据：sys_common_status、sys_menu_type、sys_data_scope、sys_oper_type
  - 涉及文件：`server/src/modules/dict/`
  - 对应需求：FR-5
- [ ] 2. 字典类型接口
  - 实现分页、增删改（编码唯一、修改编码时同步字典项、删除时级联删除字典项）
  - 对应需求：FR-1
- [ ] 3. 字典项接口
  - 实现按类型查询、增删改（同类型下值唯一）
  - 对应需求：FR-2
- [ ] 4. 按编码查询与缓存
  - [ ] 4.1 实现 `GET /api/dicts/:code/items`，使用 Redis 缓存
  - [ ] 4.2 所有写操作在事务提交后清除相关缓存
  - 对应需求：FR-3
- [ ] 5. 前端字典管理页
  - 实现类型分页表格与字典项抽屉
  - 涉及文件：`web/src/views/system/dict/`、`web/src/api/dict.ts`
  - 对应需求：FR-1、FR-2
- [ ] 6. 前端字典组件
  - [ ] 6.1 实现 `useDict` 与 `DictTag`
  - [ ] 6.2 把部门、菜单、角色、用户页面的状态显示和下拉框替换为字典组件
  - 涉及文件：`web/src/composables/useDict.ts`、`web/src/components/DictTag.vue`
  - 对应需求：FR-4
- [ ] 7. 编写测试并通过
  - [ ] 7.1 单测：缓存命中与失效、停用类型
  - [ ] 7.2 e2e：修改后立即生效
  - [ ] 7.3 前端：useDict 去重请求
  - 对应需求：FR-1 ~ FR-5
