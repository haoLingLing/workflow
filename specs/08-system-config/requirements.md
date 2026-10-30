# 系统参数配置 - 需求说明

> 来源：PRD.md § 3.8 系统参数配置

## 背景与目标
把系统名称、默认密码、登录锁定策略等可调参数放到界面上维护，修改后无需重启即可生效。

## 用户故事
- 作为系统管理员，我希望修改系统参数，以便不改代码、不重启就能调整系统行为。
- 作为系统管理员，我希望内置参数不会被误删，以便系统始终可以正常运行。

## 功能需求
### FR-1 参数维护
字段：参数名称、参数键（唯一）、参数值、是否内置、备注。

**验收标准**
- WHEN 查询参数 THEN 系统 SHALL 支持按名称、键筛选并分页
- WHEN 新增参数 THEN 系统 SHALL 校验键唯一；新增的参数均为非内置
- WHEN 编辑内置参数 THEN 系统 SHALL 只允许修改参数值和备注
- WHEN 删除内置参数 THEN 系统 SHALL 拒绝删除

### FR-2 即时生效
**验收标准**
- WHEN 参数被修改 THEN 系统 SHALL 刷新缓存，之后读取该参数的业务立即使用新值，无需重启

### FR-3 内置参数
**验收标准**
- WHEN 系统初始化 THEN 系统 SHALL 写入以下内置参数：

| 键 | 默认值 | 用途 |
| --- | --- | --- |
| sys.name | 后台管理系统 | 系统名称（登录页、顶栏） |
| sys.logo | （空） | Logo 地址 |
| user.defaultPassword | 环境变量 `USER_DEFAULT_PASSWORD` | 新建或重置用户时的密码 |
| login.maxFailCount | 5 | 连续失败锁定次数 |
| login.lockMinutes | 15 | 锁定时长（分钟） |

- WHEN 修改 `login.*` 或 `user.defaultPassword` 的值 THEN 系统 SHALL 校验格式（正整数；默认密码符合密码规则）

### FR-4 接入已有模块
**验收标准**
- WHEN 02-auth 判断锁定 THEN 系统 SHALL 读取 `login.maxFailCount` 和 `login.lockMinutes`
- WHEN 06-user 新增或重置密码 THEN 系统 SHALL 读取 `user.defaultPassword`
- WHEN 前端加载登录页和布局 THEN 系统 SHALL 通过公开接口获取 `sys.name` 和 `sys.logo`

## 非功能需求
- 读取参数时命中缓存 < 5ms

## 范围外
- 参数修改历史与回滚（修改记录由 09-audit-log 的操作日志覆盖）

## 依赖
- 02-auth、05-role-permission、06-user

## 待确认问题
- 默认密码以明文存在参数表中，界面上是否需要掩码显示？（假设显示为掩码，编辑时可以查看）
