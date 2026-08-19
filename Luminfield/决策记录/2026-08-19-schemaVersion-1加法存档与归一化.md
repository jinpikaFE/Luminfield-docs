---
project: Luminfield
graph_namespace: luminfield
type: decision
status: accepted
date: 2026-08-19
last_verified: 2026-08-19
tags:
  - luminfield/项目
  - luminfield/记录/决策
  - luminfield/存档
---

# schemaVersion 1 加法存档与归一化

## 背景

阶段 A 到阶段 C 持续增加天气、货箱、储物、委托、NPC、星邮、事件、施工、加工机和技能。如果每个增量都清档或提高版本而不提供迁移，长期试玩进度无法保留。

## 决策

1. 当前继续使用 `GameSaveV1` 与 `schemaVersion: 1`。
2. 新状态优先作为带安全默认值的加法字段加入。
3. 读取顺序固定为反序列化、版本检查、统一归一化、领域恢复。
4. 旧字段迁移和未知 ID 过滤集中在 `SaveService.Normalize` 与各系统 `NormalizeSave`。
5. 未来版本高于当前支持值时返回 `Unsupported`，不猜测降级。
6. 损坏文件保留为 `.broken-时间戳`；正常写入通过临时文件原子替换。

## 原因

- 保留玩家已有日期、库存、农田、任务、探索和关系状态。
- 迁移规则集中、可测试并能重复执行。
- 新字段缺失时有明确默认值，不需要在场景层补丁。
- 损坏或未知版本不会静默覆盖原始数据。

## 后果

- 每个新持久化字段必须补 Capture、Restore、Normalize 和旧档测试。
- 当 v1 无法表达兼容语义时，才通过新决策引入 v2。
- 确定性结果一旦写入存档，读取不得重新抽取。

## 关联

- [[Luminfield/模块/存档与兼容迁移|存档与兼容迁移]]
- [[Luminfield/决策记录/2026-08-19-稳定ID与数据驱动内容目录|稳定 ID 与数据驱动内容目录]]
