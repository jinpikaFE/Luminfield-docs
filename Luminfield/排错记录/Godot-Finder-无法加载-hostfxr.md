---
project: Luminfield
graph_namespace: luminfield
type: troubleshooting
status: resolved
source_branch: main
source_commit: d72f5d6bee304682237039c097816bd0d732ef80
last_verified: 2026-07-27
tags:
  - luminfield/项目
  - luminfield/记录/排错
  - luminfield/工具链
---

# Godot Finder 无法加载 hostfxr

## 症状

从 Finder 直接打开项目私有的 `Godot_mono.app` 后，Godot 提示 “Failed to load .NET runtime”，并明确指出无法加载 `hostfxr`；继续创建或编辑 C# 项目可能导致崩溃。

## 原因

.NET SDK 10.0.302 已完整安装在 `/Users/edy/.codex/tools/luminfield/dotnet`，但 Finder 启动的应用不会继承终端中为项目设置的 `DOTNET_ROOT`。原始 Godot 应用因此无法发现项目私有 SDK 中的宿主运行库。这不是 SDK 文件缺失，也不需要安装全局 .NET。

## 修复

- 仓库内新增 `scripts/open_editor_macos.sh`，在启动 Godot 前设置项目私有的 `DOTNET_ROOT` 与 `PATH`。
- 新增可双击的 `scripts/open_editor_macos.command`。
- 在 `/Users/edy/.codex/tools/luminfield/Luminfield Godot.app` 提供 Finder 启动入口。
- 未修改系统级环境变量，也未安装全局 SDK。

## 验证

- 启动脚本输出 Godot 4.7.1 .NET 版本。
- 从 Finder 打开 `Luminfield Godot.app` 后，编辑器成功加载 `Main.tscn`，不再出现 `hostfxr` 提示。
- 运行中进程确认加载 `/Users/edy/.codex/tools/luminfield/dotnet/host/fxr/10.0.6/libhostfxr.dylib`，以及同一 SDK 下的 `libhostpolicy.dylib` 和 `libcoreclr.dylib`。
- C# 编译 0 错误、0 警告，核心测试 38/38 通过。

## 使用规则

macOS 图形化启动统一使用 `Luminfield Godot.app` 或仓库内 `.command`；无界面检查应显式提供项目私有 `DOTNET_ROOT`。不要从 Finder 直接打开原始 `Godot_mono.app`。
