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
  - luminfield/架构
---

# GameSession 单一业务入口与 Core/Game 依赖方向

## 背景

农业、库存、世界、NPC、经济和存档会持续扩展。如果场景节点各自持有或组合规则，读档、跨日、预览和 UI 会逐渐形成多份真相，并使跨系统动作无法保证一致性。

## 决策

1. `GameSession` 是唯一全局业务入口，持有并组合所有领域系统。
2. 单系统规则放在 `src/Core/` 对应领域对象；跨系统业务顺序由 `GameSession` 编排。
3. `src/Game/` 只转发输入、订阅变化并投影世界、室内和 UI。
4. 依赖方向固定为 `Game → Core`；`Core` 不依赖 Godot 场景节点。
5. 场景不得维护第二份可写业务状态，也不得直接修改存档 DTO 或领域集合。

## 原因

- 领域测试可以脱离 Godot 场景运行。
- 存档 Capture/Restore、跨日和事件通知只有一个编排位置。
- 预览与动作能够读取同一个当前状态。
- 新场景不会自然演化成新的业务入口。

## 后果

- `GameSession` 可能较大，但它只负责组合和顺序，具体规则仍下沉到领域系统。
- 新功能必须先确定状态所有者，再接入会话与场景投影。
- 任何反向依赖或场景旁路都视为架构变更，需要单独决策。

## 关联

- [[Luminfield/模块/架构总览|架构总览]]
- [[Luminfield/模块/状态所有权与依赖方向|状态所有权与依赖方向]]
