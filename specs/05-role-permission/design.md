# 角色与权限 - 技术设计

## 概述
`RoleModule` 负责角色与授权数据；`PermissionService` 计算并缓存用户的权限集合和数据范围；全局 `PermissionGuard` 配合 `@RequirePermission()` 装饰器完成接口鉴权；`DataScopeService` 为查询构建器追加数据范围条件。

## 架构 / 流程
```mermaid
flowchart LR
  Req[请求] --> JG[JwtAuthGuard] --> PG[PermissionGuard]
  PG -->|getUserAuthz| PS[PermissionService]
  PS -->|命中| Cache[(Redis user:authz:{uid})]
  PS -->|未命中| DB[(sys_user_role / sys_role / sys_role_menu / sys_menu)]
  Svc[业务 Service] -->|applyDataScope| DS[DataScopeService] --> PS
```

## 数据模型
`sys_role`（继承 BaseEntity）：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| name | varchar(30) | 角色名称 |
| code | varchar(50) unique | 角色编码 |
| description | varchar(255) null | 描述 |
| data_scope | tinyint | 1 全部 / 2 本部门及下级 / 3 仅本部门 / 4 仅本人 |
| sort | int default 0 | 排序号 |
| status | tinyint | 1 启用 / 0 停用 |

`sys_role_menu`：`role_id`、`menu_id`，联合主键。
`sys_user_role`：`user_id`、`role_id`，联合主键，`role_id` 建索引。

缓存 `user:authz:{userId}`（JSON，TTL 1 天）：
```json
{ "roles": ["dept_admin"], "permissions": ["system:user:list"], "dataScope": 2, "deptId": 5 }
```

## 接口设计
| 方法 | 路径 | 权限标识 | 说明 |
| --- | --- | --- | --- |
| GET | /api/roles | system:role:list | 分页列表 |
| GET | /api/roles/options | 登录即可 | 启用角色下拉 |
| GET | /api/roles/:id | system:role:query | 详情（含 `menuIds`） |
| POST | /api/roles | system:role:add | 新增 |
| PUT | /api/roles/:id | system:role:edit | 编辑基本信息与数据范围 |
| PUT | /api/roles/:id/menus | system:role:edit | 覆盖保存菜单授权 `{ menuIds: [] }` |
| PUT | /api/roles/:id/status | system:role:edit | 启用/停用 |
| DELETE | /api/roles/:id | system:role:remove | 删除 |

错误码：40908 角色编码重复；40909 角色已分配给用户；40302 无权限访问（PermissionGuard 返回）。

关键方法：
- `PermissionService.getUserAuthz(userId)`：读缓存，未命中时从数据库计算（只计算启用的角色，超级管理员 `permissions = ["*:*:*"]`、`dataScope = 1`）。
- `PermissionService.evictByRole(roleId)`：查出该角色下的全部用户，删除他们的缓存。
- `PermissionService.evictUser(userId)`：供 06 调整用户角色或部门时调用。
- `DataScopeService.apply(qb, user, { deptAlias, userAlias })`：
  - 1 全部：不追加条件
  - 2 本部门及下级：`dept_id IN (getSubtreeIds(deptId))`
  - 3 仅本部门：`dept_id = deptId`
  - 4 仅本人：`user.id = userId` 或 `created_by = userId`
- 04-menu 的 `/api/menus/routes` 改为按 `permissions` 对应的菜单过滤；`/api/auth/me` 补充 `roles`、`permissions`。

## 前端 / 交互
- `views/system/role/index.vue`：分页表格；行操作"编辑 / 分配权限 / 删除 / 启停开关"。
- 分配权限抽屉：`el-tree`（show-checkbox），提供"展开/折叠"和"全选/全不选"；提交时合并 `getCheckedKeys()` 与 `getHalfCheckedKeys()`。
- 数据范围在编辑弹窗中以单选框展示。

## 异常处理
- 菜单授权保存放在事务中：先删后插。
- 缓存失效在事务提交后执行，避免读到旧数据又写回缓存。
- Redis 不可用时 `getUserAuthz` 直接查库（降级，记录告警日志）。

## 测试策略
- 单测：多角色权限并集、停用角色被忽略、数据范围取最大值、`DataScopeService` 各分支生成的条件。
- e2e：无权限访问返回 403；修改角色权限后同一令牌的下一次请求即生效；删除已分配角色失败。
