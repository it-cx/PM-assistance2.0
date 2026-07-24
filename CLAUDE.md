# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

这不是传统代码库，而是一个**以 Claude Code 为运行核心的产品经理（PM）工作空间**。仓库本身几乎没有可执行代码——它的"产物"是 Markdown 文档（PRD、竞品分析、审查报告、Issue 拆分）和自包含的 HTML 原型。所有工作通过 **agents + skills + 项目文档** 三层协作产生。

因此没有 build / lint / test 命令。"运行"一个原型 = 用浏览器直接打开对应的 `.html` 文件。

## 接手任一项目前（必做）

项目的核心要素不会自动加载。接手 `projects/<项目名>/` 下任何项目、产出任何文档或原型**之前**，必须按顺序读取该项目的：

1. `STATUS.md` —— 项目阶段、健康度、阻塞项（一眼判断项目状态）
2. `.claude/memory/MEMORY.md` —— 项目记忆索引，含 **进度** 与 **待确认项** 入口
3. `.claude/memory/reference/document-index.md` —— 文档清单与"阅读顺序建议"
4. 据上述索引深入：**项目背景与定位 → PRD 关键决策 → 决策日志 → Issue 依赖映射**

不得在未读取项目记忆的情况下直接动笔。若目标项目尚无 `.claude/memory/`，先补齐记忆骨架再开工。

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
├── STATUS.md                   # 项目状态：阶段、健康度、阻塞项（新会话快速入口）
├── <文档>.md                   # PRD / 竞品分析 / 审查报告 / Issue 拆分
├── prototypes/                 # HTML 原型（proto-expert 输出到这里）
├── *.html                      # 数据看板等独立页面
└── .claude/memory/             # 该项目专属记忆（不是全局记忆）
    ├── MEMORY.md               # 项目记忆索引（新会话入口）
    ├── project/                # progress / pending-items / prd-decisions / decisions / issue-map ...
    └── reference/              # document-index / industry-context ...
```

根目录 `PROJECTS.md` 是多项目看板，一览所有项目的阶段、健康度和阻塞项。

产出命名与落盘规范（写死在 agent 定义里）：
- 文档：Markdown，开头标版本号 + 创建日期；待确认项统一用 `[待确认]` 标记。
- 原型：纯 HTML + CSS + 原生 JS，无外部依赖（Tailwind CDN 可用）；命名 `proto-<功能名>-<版本>.html`，存到 `projects/<项目名>/prototypes/`；必须覆盖 初始/加载/空/正常/错误/边界 六态。
- PRD 功能需求标 P0/P1/P2 优先级；验收标准用 Given-When-Then。

## 审查分级标准（P0-P3）

| 等级 | 含义 | 交付决策 |
|------|------|----------|
| **P0** | 阻断性——逻辑矛盾、安全漏洞、数据丢失风险、核心流程不可用 | 必须修复才能交付 |
| **P1** | 高优先级——体验严重受损、主要场景遗漏、关键边界未处理 | 强烈建议修复，需有明确理由才能放行 |
| **P2** | 中优先级——次要场景问题、文案/提示不够清晰、交互不够顺畅 | 建议修复，可接受在后续迭代中跟进 |
| **P3** | 改进建议——锦上添花的优化、风格一致性微调 | 可选修复，不阻塞交付 |

## 项目管理

### 项目状态（STATUS.md）

每个 `projects/<项目名>/` 下必须有 `STATUS.md`，是判断项目状态的最快入口。`projects/_template/STATUS.md` 提供完整模板，新建项目时复制并填写。

```yaml
phase: define | discover | prototype | review | develop | shipped | paused | archived
health: green | yellow | red
last_updated: <YYYY-MM-DD>
blockers: []
next_milestone: "<一句话描述>"
```

**阶段流转**：`define` → `discover`（可选）→ `prototype` → `review` → `develop` → `shipped`。任何时候可转入 `paused` 或 `archived`。

**health 判定规则**：
- 🟢 `green`：按计划推进，无阻塞项
- 🟡 `yellow`：[待确认] 堆积 > 5 条 / 超 2 周无进展 / 有未解决 P1 / 里程碑逾期未更新
- 🔴 `red`：有未解决 P0 / 关键依赖缺失 / 超 4 周无进展

STATUS.md 的变更日志记录每次 phase 或 health 变化。

### 多项目看板（PROJECTS.md）

仓库根目录 `PROJECTS.md` 汇总所有项目状态。每次项目 STATUS.md 更新后，同步更新 PROJECTS.md 中对应行。Claude 在会话开始时可先读此文件快速了解全局。

### 新建项目 Checklist

创建 `projects/<新项目>/` 时，按以下步骤初始化：

1. 复制 `projects/_template/` 到 `projects/<新项目>/`
2. 填写 `STATUS.md`：设置 `phase: define`、`health: green`、`created` 日期
3. 填写 `.claude/memory/MEMORY.md` 中的项目名
4. 填写 `.claude/memory/project/background.md`：定位、用户、目标、范围边界
5. 在 `PROJECTS.md` 表格中新增一行

### 阶段门禁（Gate）

三专家流水线每个交接点设有 gate，由 **交接方** 确认条件满足后放行：

**Gate 1：doc → proto（需求冻结，进入原型）**
- [ ] P0 问题清零
- [ ] 所有 `[待确认]` 项已决议或标注"原型阶段不阻塞"
- [ ] PRD 功能需求优先级（P0/P1/P2）已标注
- [ ] 核心用户故事验收标准（Given-When-Then）已覆盖

**Gate 2：proto → review（原型送审）**
- [ ] 原型覆盖六态：初始 / 加载中 / 空数据 / 正常 / 错误 / 边界
- [ ] 所有可交互元素有反馈（点击、输入、hover）
- [ ] `prototype-audit` 像素级审计通过（`pass: true`）

**Gate 3：review → develop（审查通过，移交工程）**
- [ ] P0 问题清零
- [ ] P1 问题已修复或有明确跟进计划
- [ ] review-expert 出具放行结论（`gate: pass`）
- [ ] `STATUS.md` phase 更新为 `develop`

### 决策日志（decisions.md）

项目的关键决策必须记录在 `.claude/memory/project/decisions.md`，格式：

```markdown
## D001 | <YYYY-MM-DD> | <一句话标题>
**背景**：<为什么需要做这个决策>
**选项**：A) ...  B) ...  C) ...
**决定**：<选了哪个，为什么>
**影响**：<对后续工作的影响>
```

决策按 D001, D002... 递增编号。适用于技术选型、范围裁剪、交互方案取舍等"三个月后回来继续做时需要知道为什么"的场景。

## 记忆系统（两层，务必分清）

- **全局记忆** `C:\Users\Administrator\.claude\projects\D--PM-assistance\memory\`——只放"关于用户本人"和跨项目的偏好/工作方式。
- **项目记忆** `projects/<项目名>/.claude/memory/`——放某个产品的进度、决策、背景、文档索引。产品相关的一切写这里，**不要污染全局记忆**。

新会话进入某项目时的上手路径：先读该项目 `.claude/memory/MEMORY.md`，再按 `reference/document-index.md` 的"阅读顺序建议"深入。

## Git 约定

- 远程 `origin` = https://github.com/it-cx/PM-assistance2.0.git，主分支 `master`。
- commit message 用中文，`feat: <简述>` 风格（见历史提交）。
- `.gitignore` 已排除 `.claude/jobs/`、`.claude/worktrees/`、`.claude/plans/`、`.claude/scheduled_tasks.json` 等运行时目录——这些不入库。
