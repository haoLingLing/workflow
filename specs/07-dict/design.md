# 数据字典 - 技术设计

## 概述
`DictModule` 维护两张表，按编码查询时走 Redis 缓存（`dict:{code}`）；任何写操作在事务提交后删除对应编码的缓存。

## 架构 / 流程
```mermaid
flowchart LR
  W[useDict] -->|GET /api/dicts/:code/items| C[DictController]
  C --> S[DictService]
  S -->|命中| R[(Redis dict:{code})]
  S -->|未命中| D[(sys_dict_type / sys_dict_data)]
```

## 数据模型
`sys_dict_type`：name varchar(50)、code varchar(100) unique、status tinyint、remark varchar(255) null。

`sys_dict_data`：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| dict_code | varchar(100) | 所属类型编码（索引） |
| label | varchar(100) | 标签 |
| value | varchar(100) | 值，`(dict_code, value)` 唯一 |
| sort | int default 0 | 排序号 |
| tag_type | varchar(20) null | Element Plus tag 类型：primary/success/warning/danger/info（假设） |
| status | tinyint | 1 启用 / 0 停用 |

修改字典类型的编码时，同步更新其字典项的 `dict_code`，并清除新旧两个编码的缓存。

## 接口设计
| 方法 | 路径 | 权限标识 | 说明 |
| --- | --- | --- | --- |
| GET | /api/dict-types | system:dict:list | 分页 |
| POST / PUT / DELETE | /api/dict-types[/:id] | system:dict:add / edit / remove | 类型维护 |
| GET | /api/dict-types/:code/data | system:dict:list | 某类型下的全部字典项（含停用） |
| POST / PUT / DELETE | /api/dict-data[/:id] | system:dict:add / edit / remove | 字典项维护 |
| GET | /api/dicts/:code/items | 登录即可 | 启用的字典项 `[{ label, value, tagType }]` |

错误码：40911 字典编码重复；40912 同类型下字典值重复。

## 前端 / 交互
- `views/system/dict/index.vue`：字典类型分页表格；点击某类型打开右侧抽屉，维护其字典项。
- `composables/useDict.ts`：`const { sys_common_status } = useDict('sys_common_status')`，返回响应式数组；模块级 Map 缓存 Promise，避免重复请求。
- `components/DictTag.vue`：`<DictTag :options="..." :value="row.status" />`。
- 把 03–06 页面中的状态显示和下拉框替换为字典组件。

## 异常处理
- 缓存写入失败只记录告警，不影响返回结果。
- 删除类型和字典项在同一事务中完成。

## 测试策略
- 单测：缓存命中与失效、停用类型返回空数组。
- e2e：修改字典项后立即可以查到新数据。
- 前端：`useDict` 同一编码只请求一次。
