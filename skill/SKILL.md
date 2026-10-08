---
name: qsc-qplug-dev
description: "Q-SYS qplug 插件开发技能。当用户要求创建、编辑、调试 Q-SYS Designer 插件（.qplug 文件），或涉及 wxl_personal_plug 系列插件时使用。覆盖控件定义、面板布局、属性参数、运行时 Lua 逻辑、GitHub 发布。Use when creating, editing, or debugging Q-SYS Designer plugins (.qplug), or working with the wxl_personal_plug plugin series."
---

# Q-SYS Qplug Development

## Overview

Q-SYS qplug 是 Q-SYS Designer 的 Lua 插件，单文件包含元信息、属性、控件、布局和运行时逻辑。本技能提供编写规范、控件类型参考、常见陷阱和调试经验。

## 快速参考 / Quick Reference

**编写规范与控件类型**: 读取 `references/qplug-standards.md`
- 9 文件 Plugin Compiler 框架结构
- 必需函数（GetControls/GetProperties/GetControlLayout/GetPages 等）
- Button/Knob/Text/Indicator 控件类型与引脚规则
- 布局图形类型与坐标规范
- 实践验证的常见陷阱表

## 工作流 / Workflow

1. **需求整理**: 先列出输入/输出引脚、属性参数、面板页面、行为逻辑，与用户确认
2. **编写 qplug**: 单文件结构，按 PluginInfo → GetProperties → RectifyProperties → GetPages → GetControls → GetControlLayout → 运行时 顺序
3. **归档**: 复制到 `C:\Users\<user>\Documents\QSC\Q-Sys Designer\Plugins\`
4. **测试**: Q-SYS Designer 中拖入组件，检查引脚、面板、运行时日志
5. **发布**: 每个模块独立 GitHub 仓库，中英文双语 README

## 关键规则 / Critical Rules (必须遵守)

- `ButtonType="Trigger"` 的 EventHandler 中 `c.Boolean` 为 false，必须无条件执行
- Knob 必须设 `ControlUnit="Integer"` 才生成外部引脚
- `ButtonStyle` 只影响面板外观，不影响引脚类型
- 修改 `*_count` 属性后必须删除并重新拖入组件
- 面板所有框留足边距，宁大勿小
- 每个插件底部必须有 Notes 中英文说明
- 面板文字全英文，Notes 中英文双语
- 看门狗用 `os.time()`，不要用 `os.clock()`

## 资源 / Resources

- `references/qplug-standards.md` — 完整编写规范与陷阱表
- 官方文档: https://help.qsys.com/DeveloperHelp/
- Plugin Compiler: https://help.qsys.com/DeveloperHelp/#Development_Tools/Plugin_Compiler.htm
