# 菜单管理 - 任务清单

> 状态说明：`[ ]` 未完成，`[✅]` 已完成

- [ ] 1. 菜单表与种子数据
  - [ ] 1.1 创建 `MenuEntity` 与迁移脚本
  - [ ] 1.2 编写幂等种子脚本：系统管理目录及各模块菜单、按钮权限标识
  - 涉及文件：`server/src/modules/menu/menu.entity.ts`、`server/src/database/seeds/menu.seed.ts`
  - 对应需求：FR-4
- [ ] 2. 菜单增删改查接口
  - [ ] 2.1 实现菜单树查询（筛选）、精简树、详情
  - [ ] 2.2 实现新增/编辑，含按类型的字段校验与权限标识唯一、格式校验
  - [ ] 2.3 实现删除，含子节点校验并预留角色关联清理
  - 涉及文件：`server/src/modules/menu/`
  - 对应需求：FR-1
- [ ] 3. 当前用户路由接口
  - 实现 `GET /api/menus/routes`：过滤按钮和停用节点，转换为路由结构；暂时超级管理员返回全部，并预留按用户过滤的扩展点
  - 对应需求：FR-2
- [ ] 4. 前端菜单管理页
  - [ ] 4.1 实现树形表格与筛选
  - [ ] 4.2 实现按类型动态显示字段的表单与 `IconSelect` 图标选择器
  - 涉及文件：`web/src/views/system/menu/`、`web/src/components/IconSelect.vue`、`web/src/api/menu.ts`
  - 对应需求：FR-1
- [ ] 5. 前端动态路由
  - [ ] 5.1 实现 `router/dynamic.ts` 组件映射与 404 回退
  - [ ] 5.2 实现 `stores/permission.ts`，路由守卫中加载路由并注册，退出时重置
  - [ ] 5.3 侧边栏改为基于路由数据渲染（隐藏 `hidden` 节点）
  - 对应需求：FR-2、FR-3
- [ ] 6. 按钮权限指令
  - 实现 `v-permission` 指令（支持数组，`*:*:*` 拥有全部权限）
  - 涉及文件：`web/src/directives/permission.ts`
  - 对应需求：FR-3
- [ ] 7. 编写测试并通过
  - [ ] 7.1 单测：字段校验、权限标识校验、路由树构建
  - [ ] 7.2 前端：组件映射回退、`v-permission`
  - [ ] 7.3 e2e：删除有子节点的菜单失败
  - 对应需求：FR-1 ~ FR-4
