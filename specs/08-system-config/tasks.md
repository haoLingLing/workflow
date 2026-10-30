# 系统参数配置 - 任务清单

> 状态说明：`[ ]` 未完成，`[✅]` 已完成

- [ ] 1. 参数表与种子数据
  - 创建 `SysConfigEntity` 与迁移脚本，并写入 5 个内置参数
  - 涉及文件：`server/src/modules/sys-config/`
  - 对应需求：FR-3
- [ ] 2. 参数维护接口
  - [ ] 2.1 实现分页、详情、新增（键唯一）
  - [ ] 2.2 实现编辑（内置参数只能改值和备注，值需经过校验注册表）
  - [ ] 2.3 实现删除（内置参数拒绝）
  - 对应需求：FR-1、FR-3
- [ ] 3. 读取服务与缓存
  - 实现 `SysConfigService.get / getNumber`，使用 Redis 缓存，写操作后清除
  - 对应需求：FR-2
- [ ] 4. 接入已有模块
  - [ ] 4.1 02-auth 的锁定逻辑改为读取 `login.maxFailCount` 和 `login.lockMinutes`
  - [ ] 4.2 06-user 的新增和重置密码改为读取 `user.defaultPassword`
  - [ ] 4.3 实现 `GET /api/configs/public`，前端登录页和顶栏显示系统名称和 Logo
  - 对应需求：FR-4
- [ ] 5. 前端参数管理页
  - 实现分页表格与编辑弹窗（内置参数标签、默认密码掩码）
  - 涉及文件：`web/src/views/system/config/`、`web/src/api/config.ts`
  - 对应需求：FR-1
- [ ] 6. 编写测试并通过
  - [ ] 6.1 单测：内置参数保护、值校验、缓存失效
  - [ ] 6.2 e2e：修改锁定次数后立即生效；修改默认密码后新建用户使用新值
  - 对应需求：FR-1 ~ FR-4
