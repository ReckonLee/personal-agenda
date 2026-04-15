# 个人日程秘书

> 基于 LLM Wiki 模式的 AI 驱动个人任务管理系统，无需安装任何软件，双击 HTML 即可查看可交互看板。

---

## 目录

- [个人日程秘书](#个人日程秘书)
  - [目录](#目录)
  - [系统概述](#系统概述)
  - [快速开始](#快速开始)
    - [环境要求](#环境要求)
    - [第一次使用](#第一次使用)
  - [目录结构](#目录结构)
  - [使用方法](#使用方法)
    - [新增任务（Ingest）](#新增任务ingest)
    - [更新任务状态（Update）](#更新任务状态update)
    - [刷新看板（View）](#刷新看板view)
  - [看板功能](#看板功能)
  - [历史归档](#历史归档)
  - [分享给其他人使用](#分享给其他人使用)
  - [任务文件格式参考](#任务文件格式参考)

---

## 系统概述

本系统将 AI 大语言模型作为"个人秘书"，通过对话完成所有任务管理操作：

- 用自然语言描述待办事项 → AI 自动结构化写入系统
- 说「XX 做完了」→ AI 更新状态
- 打开 `views/dashboard.html` → 可视化看板，支持状态切换、子任务勾选、备注编辑，**无服务器、零配置**

核心设计原则：**AI 只记录任务，不执行任务**（除非明确说"帮我做 XX"）。

---

## 快速开始

### 环境要求

- VS Code + [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) 扩展
- 将本项目文件夹作为 VS Code 工作区打开

### 第一次使用

1. 打开 VS Code，在 Chat 窗口输入：

   ```
   这是我今天要做的事：[粘贴你的待办内容]
   ```

2. Copilot 会自动创建任务文件并更新看板
3. 双击 `views/dashboard.html` 在浏览器中查看

---

## 目录结构

```
PersonalAgenda/
├── .github/
│   ├── copilot-instructions.md   # AI 工作规则（Schema）
│   └── skills/
│       └── personal-agenda/
│           └── SKILL.md          # 本技能定义文件
├── inbox/                        # 原始输入暂存（聊天记录、截图备注）
├── tasks/                        # 任务文件（每个任务一个 .md）
│   └── YYYYMMDD-中文描述.md
├── views/
│   ├── dashboard.html            # 可交互看板（数据内嵌）
│   ├── tasks.js                  # 历史归档数据（只读，AI 维护）
│   └── tasks.json                # 历史归档 JSON 备份
├── index.md                      # 任务总索引
└── log.md                        # 操作历史（仅追加）
```

---

## 使用方法

### 新增任务（Ingest）

**告诉 AI 你有什么要做的事**，支持三种方式：

**方式一：直接口述**

```
帮我记录几个待办：
- 下周五之前完成xxxxx的xxxxxxxxxx模块功能改进
- xxxxx的开发
```

**方式二：粘贴聊天记录**

```
处理 inbox，这是今天的工作群消息：
[粘贴聊天记录内容]
```

**方式三：将文件放入 inbox/**
将聊天记录截图或文本文件拖入 `inbox/` 文件夹，然后对 AI 说：

```
处理 inbox
```

AI 会自动：

- 识别所有待办事项
- 推断截止时间和优先级
- 创建结构化任务文件
- 更新看板数据
- 列出识别结果请你确认

---

### 更新任务状态（Update）

```
xxxxx调试做完了

xxxxx任务取消

xxxxx加个备注：等甲方提供最新资料
```

AI 会更新对应任务文件并同步看板。

---

### 刷新看板（View）

当你在看板外手动修改了任务文件，或想重新生成完整看板时：

```
更新看板
```

或

```
我本周完成了什么？
```

---

## 看板功能

双击 `views/dashboard.html` → 浏览器打开，无需服务器。

| 功能           | 说明                                                                   |
| -------------- | ---------------------------------------------------------------------- |
| **分区展示**   | 今日任务 / 本周任务 / 本月任务 / 以后(Someday) / 已完成                |
| **状态切换**   | 每张卡片底部有「▶ 进行中 / ✓ 已完成 / ✕ 已取消」三个按钮，当前状态高亮 |
| **子任务勾选** | 任务的每个子功能点渲染为独立 checkbox，可单独勾选，文本可复制          |
| **备注编辑**   | 每张卡片有全宽备注输入框，实时编辑                                     |
| **自动保存**   | 所有操作自动保存到浏览器 localStorage，刷新不丢失                      |
| **重新加载**   | 点击「↻ 重新加载」合并 AI 更新的最新任务数据                           |
| **历史归档**   | 点击「📚 历史」查看两个月前已完成的归档任务                            |
| **深色模式**   | 跟随系统自动切换亮色/暗色主题                                          |

**优先级颜色**：🔴 urgent / 🟠 high / 🟢 normal / ⚫ low

---

## 历史归档

为避免 HTML 文件随时间增大，系统在每次操作时自动归档旧数据：

- **归档规则**：`completed_date` 早于「当前月往前推两个月的1日」的已完成/取消任务
  - 例：当前为 2026-04，阈值为 2026-02-01，2月以前完成的任务自动归档
- **归档位置**：`views/tasks.js` 和 `views/tasks.json`（由 AI 维护，浏览器不写入）
- **查看方式**：点击看板右上角「📚 历史」按钮，弹出模态框按年月折叠展示

---

## 分享给其他人使用

本系统通过 `.github/copilot-instructions.md` 驱动 AI 行为，其他人只需：

1. 将整个项目文件夹用 VS Code 打开
2. 安装 GitHub Copilot 扩展
3. `copilot-instructions.md` 会被 Copilot 自动加载，无需任何配置

如果对方需要全新的空白系统（不含你的任务数据），保留以下文件即可：

- `.github/copilot-instructions.md`
- `.github/skills/personal-agenda/SKILL.md`
- `views/dashboard.html`（清空 `TASKS_DATA` 中的 tasks 数组）
- `views/tasks.js` / `views/tasks.json`（空归档）
- `log.md`、`index.md`（清空内容）

---

## 任务文件格式参考

```yaml
---
id: 20260415-xxxxx-xxxxx改进
title: xxxxx — xxxxxxxxxx模块多项功能改进
created: 2026-04-15 10:00
due: 2026-04-17 17:00
category: week # today | week | month | someday
priority: high # urgent | high | normal | low
status: pending # pending | done | cancelled
completed_date:
tags: [xxxxx, xxxxx, xxxxx]
notes: "模块通用性：是 · 开发：xxxxx"
---
```

`detail` 字段（子任务列表）示例：

```yaml
detail:
  - xxxxx增加退回上一级功能
  - xxxxx增加检索条件
  - xxxxx限制改为 500 条
```
