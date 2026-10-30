# 登录与认证 - 技术设计

## 概述
后端 `AuthModule` 负责验证码、登录、刷新、退出和改密，`JwtAuthGuard` 作为全局守卫完成鉴权。令牌使用 JWT，失效控制依赖 Redis：单个令牌退出走黑名单，用户级强制下线走 `tokenVersion`。本模块先建立 `sys_user` 的核心字段，06-user 在此基础上补全管理功能。

## 架构 / 流程
```mermaid
sequenceDiagram
  participant W as Web
  participant A as AuthController
  participant R as Redis
  participant D as MySQL
  W->>A: GET /auth/captcha
  A->>R: SET captcha:{id} code EX 120
  W->>A: POST /auth/login {username,password,captchaId,captcha}
  A->>R: GETDEL captcha:{id} 校验
  A->>R: GET login:lock:{username}
  A->>D: 查询用户 + bcrypt.compare
  alt 失败
    A->>R: INCR login:fail:{username}（达到阈值则写 lock）
  else 成功
    A->>R: DEL login:fail:{username}; SET refresh:{uid}:{jti}
    A-->>W: accessToken, refreshToken, mustChangePassword
  end
```

## 数据模型
`sys_user`（本模块建立核心字段，继承 BaseEntity）：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| username | varchar(32) unique | 账号，创建后不可改 |
| password_hash | varchar(100) | bcrypt 哈希 |
| real_name | varchar(32) | 姓名 |
| phone | varchar(20) null | 手机号 |
| email | varchar(64) null | 邮箱 |
| avatar | varchar(255) null | 头像地址 |
| dept_id | bigint null | 所属部门（03-department 外键） |
| status | tinyint | 1 启用 / 0 禁用 |
| is_builtin | tinyint | 1 内置超级管理员 |
| must_change_password | tinyint | 1 需修改密码 |
| token_version | int default 0 | 用户级令牌版本，+1 即全部失效 |
| deleted_at | datetime null | 逻辑删除 |

Redis 键：

| 键 | 值 | TTL |
| --- | --- | --- |
| `captcha:{captchaId}` | 验证码文本（小写） | 120s |
| `login:fail:{username}` | 连续失败次数 | 锁定时长 |
| `login:lock:{username}` | 1 | 锁定时长（默认 900s） |
| `token:black:{jti}` | 1 | 访问令牌剩余有效期 |
| `refresh:{userId}:{jti}` | 1 | 7d |
| `user:tv:{userId}` | 当前 token_version（缓存） | 无 |

JWT 载荷：`{ sub: userId, username, tv: tokenVersion, jti, typ: "access"|"refresh" }`。

## 接口设计
| 方法 | 路径 | 鉴权 | 说明 |
| --- | --- | --- | --- |
| GET | /api/auth/captcha | Public | 返回 `{ captchaId, image }`（image 为 svg data URL） |
| POST | /api/auth/login | Public | 请求 `{ username, password, captchaId, captcha }`，返回 `{ accessToken, refreshToken, expiresIn, mustChangePassword }` |
| POST | /api/auth/refresh | Public | 请求 `{ refreshToken }`，返回新的一对令牌（旧刷新令牌失效） |
| POST | /api/auth/logout | 登录 | 拉黑当前访问令牌，删除对应刷新令牌 |
| GET | /api/auth/me | 登录 | 返回当前用户基本信息；`roles`、`permissions` 由 05-role-permission 补充 |
| PUT | /api/auth/password | 登录 | 请求 `{ oldPassword, newPassword }` |

错误码：

| code | 含义 |
| --- | --- |
| 40001 | 验证码错误或已过期 |
| 40002 | 密码不符合规则 |
| 40101 | 账号或密码错误 |
| 40102 | 账号已锁定，message 含剩余分钟数 |
| 40103 | 账号已禁用 |
| 40104 | 令牌无效或已过期 |
| 40301 | 需先修改密码 |

`AuthService.revokeAllTokens(userId)`：`token_version + 1` 并更新 `user:tv:{userId}`，删除 `refresh:{userId}:*`。供 06-user 使用。

`JwtAuthGuard` 校验顺序：签名与过期 → `typ === "access"` → `token:black:{jti}` 不存在 → `tv` 与 `user:tv:{userId}` 一致 → 若 `must_change_password` 则只放行改密与退出。

登录成功/失败时通过 `EventEmitter2` 发出 `auth.login` 事件：`{ username, userId?, success, reason?, ip, userAgent }`。

## 前端 / 交互
- `views/login/index.vue`：账号、密码、验证码（点击图片刷新）；失败后自动刷新验证码。
- `stores/auth.ts`：保存令牌（localStorage）、用户信息；`login`、`logout`、`refresh` 动作。
- `request.ts` 增强：注入 `Authorization: Bearer`；401 时用单例 Promise 刷新，期间的请求排队，刷新成功后重放，失败则清空并跳转 `/login?redirect=`。
- 路由守卫：未登录跳登录页；`mustChangePassword` 时强制跳 `/change-password`。
- `views/change-password/index.vue`：旧密码、新密码、确认密码，前端同步校验规则。

## 异常处理
- 用户不存在时同样执行一次 bcrypt 比较（假哈希），避免时序差异泄露账号是否存在。
- 锁定期间不再累加失败次数。
- Redis 不可用时登录接口返回 500，不降级跳过锁定校验。

## 测试策略
- 单测：密码规则校验；失败计数与锁定逻辑；守卫的各种拒绝分支（黑名单、tv 不一致、mustChangePassword）。
- e2e：登录成功 → 访问 me → 退出 → 再访问返回 401；连续 5 次错误后锁定；刷新令牌轮换后旧令牌不可用。
- 前端：并发 401 只触发一次刷新。
