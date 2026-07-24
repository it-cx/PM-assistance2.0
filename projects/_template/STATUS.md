---
project: <项目名>
created: <YYYY-MM-DD>
---

# STATUS — <项目名>

```yaml
phase: define | discover | prototype | review | develop | shipped | paused | archived
health: green | yellow | red
last_updated: <YYYY-MM-DD>
blockers: []
next_milestone: "<一句话描述>"
```

## 阶段说明

| phase | 含义 |
|-------|------|
| `define` | 需求定义阶段——PRD、用户故事、竞品分析 |
| `discover` | 调研发现阶段——用户访谈、市场分析、机会评估 |
| `prototype` | 原型设计阶段——proto-expert 在产出 HTML |
| `review` | 审查迭代阶段——review-expert 审查中，等待修复 |
| `develop` | 开发交付阶段——需求已移交工程，PM 跟踪进度 |
| `shipped` | 已交付——功能上线，文档归档 |
| `paused` | 暂停——主动暂停，保留上下文 |
| `archived` | 归档——项目结束或不再维护 |

## health 信号

| health | 含义 | 典型触发条件 |
|--------|------|-------------|
| 🟢 `green` | 正常推进 | 按计划进行，无阻塞 |
| 🟡 `yellow` | 有风险 | [待确认] 项堆积 > 5 条，或超过 2 周无进展，或有未解决的 P1 问题 |
| 🔴 `red` | 阻塞 | 有未解决的 P0 问题，或关键依赖缺失，或超过 4 周无进展 |

## 变更日志

| 日期 | phase 变更 | 备注 |
|------|-----------|------|
| <YYYY-MM-DD> | → `define` | 项目创建 |
