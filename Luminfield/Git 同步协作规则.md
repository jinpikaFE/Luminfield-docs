---
project: Luminfield
graph_namespace: luminfield
type: collaboration-rule
status: active
date: 2026-08-19
tags:
  - luminfield/项目
  - luminfield/协作
  - luminfield/git
---

# Git 同步协作规则

本 Vault 使用独立 Git 仓库同步项目知识，不依赖 Obsidian Sync。Git 只负责版本协作，不提供实时多人编辑；同一篇笔记尽量一次只由一个人修改。

远程仓库：`git@github.com:jinpikaFE/Luminfield-docs.git`

## 日常流程

1. 修改前在 Vault 根目录执行 `git pull --ff-only`。
2. 修改笔记时保持小步提交；一组相关笔记改完就提交，不把多天工作堆在一个提交里。
3. 修改后执行 `git status`，确认只包含本次知识变更。
4. 提交本次变更：`git add <files>`，然后 `git commit`。
5. 提交后立即执行 `git push`，不要把已提交内容长期留在本地。

## 冲突处理

- 如果 `git pull --ff-only` 失败，先看 `git status`；有本地未提交修改时先提交或撤回自己的本地改动。
- 如果 `git push` 被拒绝，说明远端已有新提交；先执行 `git pull`，解决冲突并确认笔记内容正确后再 `git push`。
- 冲突解决时优先保留双方真实结论，不用整段覆盖他人的新增内容。

## 不提交的内容

- 不提交 `.DS_Store`、`.trash/`、`.obsidian/workspace*.json` 和缓存目录。
- 不提交临时草稿、密钥、令牌、账号凭证、私有远程地址截图或一次性日志。
- `.obsidian` 中可同步基础设置，但每个人的工作区布局不进入 Git。

## 提交信息

建议使用中文 Conventional Commit：

```text
docs(vault): 更新 Luminfield 知识同步规则

- 补充修改前 pull、修改后 commit/push 的协作流程
- 说明冲突处理和不进入 Git 的本地文件
```

## 关联

- [[Luminfield/00-项目总览|项目总览]]
- [[Luminfield/01-工作区与分支|工作区与分支]]
