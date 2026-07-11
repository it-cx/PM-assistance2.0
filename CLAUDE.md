# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

这不是传统代码库，而是一个**以 Claude Code 为运行核心的产品经理（PM）工作空间**。仓库本身几乎没有可执行代码——它的"产物"是 Markdown 文档（PRD、竞品分析、审查报告、Issue 拆分）和自包含的 HTML 原型。所有工作通过 **agents + skills + 项目文档** 三层协作产生。

因此没有 build / lint / test 命令。"运行"一个原型 = 用浏览器直接打开对应的 `.html` 文件。

## 核心架构：三专家流水线

工作由三个 subagent 串联完成，通常按此顺序调度（见 `.claude/agents/`）：

1. **doc-expert** — 产出文档：PRD、用户故事、竞品分析、Issue 拆分。掌握 38 个 PM 流程技能，按"澄清→定义→发现→基础→深化→交付"6 阶段调度。
2. **proto-expert** — 把需求转成单文件、内联 CSS/JS、可直接打开的 HTML 原型，带交互标注。掌握 24 个设计技能。
3. **review-expert** — 对抗式审查文档与原型，输出 P0/P1/P2/P3 分级问题 + 修复建议。核心技能是 `utility-pm-critic`。

**关键约定：任何 doc/proto 产出后都应交给 review-expert 对抗审查**（三个 agent 定义里都写死了这条闭环规则）。`doc-expert` 完成后用 `handoff` 交接给 `proto-expert`。

技能库位于 `.claude/skills/`（80+ 个），通过 Skill 工具按需调用；具体"什么场景调什么技能"的映射表**以三个 agent 定义文件为准**——修改工作流前先读它们，不要凭空发明技能名。

## 目录结构与产出规范

```
projects/<项目名>/              # 每个产品一个目录，所有产出落在这里
├── <文档>.md                   # PRD / 竞品分析 / 审查报告 / Issue 拆分
├── prototypes/                 # HTML 原型（proto-expert 输出到这里）
├── *.html                      # 数据看板等独立页面
└── .claude/memory/             # 该项目专属记忆（不是全局记忆）
    ├── MEMORY.md               # 项目记忆索引（新会话入口）
    ├── project/                # progress / pending-items / prd-decisions / issue-map ...
    └── reference/              # document-index / industry-context ...
```

产出命名与落盘规范（写死在 agent 定义里）：
- 文档：Markdown，开头标版本号 + 创建日期；待确认项统一用 `[待确认]` 标记。
- 原型：纯 HTML + CSS + 原生 JS，无外部依赖（Tailwind CDN 可用）；命名 `proto-<功能名>-<版本>.html`，存到 `projects/<项目名>/prototypes/`；必须覆盖 初始/加载/空/正常/错误/边界 六态。
- PRD 功能需求标 P0/P1/P2 优先级；验收标准用 Given-When-Then。

## 记忆系统（两层，务必分清）

- **全局记忆** `C:\Users\Administrator\.claude\projects\D--PM-assistance\memory\`——只放"关于用户本人"和跨项目的偏好/工作方式。
- **项目记忆** `projects/<项目名>/.claude/memory/`——放某个产品的进度、决策、背景、文档索引。产品相关的一切写这里，**不要污染全局记忆**。

新会话进入某项目时的上手路径：先读该项目 `.claude/memory/MEMORY.md`，再按 `reference/document-index.md` 的"阅读顺序建议"深入。

## Git 约定

- 远程 `origin` = https://github.com/it-cx/PM-assistance2.0.git，主分支 `master`。
- commit message 用中文，`feat: <简述>` 风格（见历史提交）。
- `.gitignore` 已排除 `.claude/jobs/`、`.claude/worktrees/`、`.claude/plans/`、`.claude/scheduled_tasks.json` 等运行时目录——这些不入库。
