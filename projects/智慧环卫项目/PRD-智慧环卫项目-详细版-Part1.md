# PRD：智慧环卫项目（详细版）— Part 1：整体架构 + 运营总览

**版本**: 2.0
**日期**: 2026-07-14
**状态**: Draft
**关联原型**:
- [产业集团运营分析](../无人环卫平台/group-analysis.html)
- [项目公司运营看板](../无人环卫平台/project-dashboard.html)

> 本文为详细版 PRD，覆盖页面交互逻辑、字段定义、数据计算公式、状态处理等实现级细节。

---

## 1. 项目概述

### 1.1 产品定义

两级数据看板体系，为福龙马集团无人环卫车运营提供数据监控与分析能力：

| 层级 | 受众 | 原型文件 | 定位 |
|------|------|----------|------|
| **产业公司层级** | 集团管理者 | `group-analysis.html` | 横向对比 5 家项目公司运营效率，按车型维度（SD15/SD22C）分析 |
| **项目公司层级** | 项目运营人员 | `project-dashboard.html` | 单项目公司的单车实时监控与日常运营管理 |

### 1.2 技术栈

- 纯 HTML + CSS + 原生 JavaScript，单文件自包含
- ECharts 5.5.0（CDN）渲染所有图表，带 try-catch 保护（ECharts 加载失败时图表区域静默跳过，不影响其他模块渲染）
- Tailwind CSS（CDN）提供基础样式工具
- 无后端依赖，数据为前端 Mock（后续接入真实 API）
- 设计系统：CITIBOT-OPS（project-dashboard，深色科技感）/ 浅色企业风（group-analysis，白底蓝调）

---

## 2. 全局组件与交互

### 2.1 group-analysis.html — Topbar 顶栏

| 组件 | DOM 元素/ID | 交互行为 |
|------|------------|---------|
| Logo | `C` 蓝色方块 | 静态展示 |
| 标题 | "CITIBOT 集团运营分析" | 静态 |
| 副标题 | "多项目公司 · 无人清扫车运营数据 · 月度分析" | 静态 |
| **车型切换** | `#globalModelToggle` | SD15 / SD22C 分段按钮组，蓝色高亮 = 当前选中。切换→ `_currentModel` 变量更新 → `switchTab(currentTab)` 重新渲染当前 tab 全部内容 |
| **月份选择器** | `#ymPicker` | 详情见 2.3 |
| 导出按钮 | `#btnExport` | 当前占位 |

### 2.2 group-analysis.html — 车型切换（全局）

- **位置**：Topbar 右侧，与月份选择器并列
- **样式**：`.model-toggle` 内联分段按钮组，边框包裹，选中态蓝底白字
- **变量**：`_currentModel = 'SD15'` 或 `'SD22C'`
- **影响范围**：四个 tab 全部内容（统计卡片标题 + 数值 + 排名表数据 + 图表数据），按车型过滤
- **过滤规则**：
  - 仅统计 `MONTHLY[name].vehicles` 中 `model === _currentModel` 的车辆
  - 没有该车型的公司不出现在表中
  - 车型聚合数据存储在 `_overviewByModel[model]`
  - 开关按钮始终在 DOM 中（不随 tab 切换销毁），通过事件委托 `document.addEventListener('click')` 监听
- **扩展性**：新增车型只需在 `['SD15', 'SD22C']` 数组后追加，topbar 按钮自动生成

### 2.3 group-analysis.html — 月份选择器

**组件结构：**
```
[📅 2026年 6月 ▼]        ← 触发器按钮 .ym-trigger
└─ 下拉面板 .ym-drop (240px宽)
   ├─ [◀] 2026年 [▶]     ← .ym-drop-head 年份行
   └─ 12 月按钮网格       ← .ym-drop-grid (4列)
```

**交互规则：**
1. 点击触发器 → 展开/收起面板（`hidden` 属性切换），展开时滚动到选中月份
2. 年份 ← → 按钮：下限 2020，上限为 `new Date().getFullYear()`
3. 月份网格：`renderYmGrid()` 遍历 1~12 月，判断 `isActive`（=当前选中月）和 `isFuture`（= 超过当前真实月份）
4. **未来月份置灰**：`color: #CBD5E1; cursor: not-allowed; pointer-events: none`，`disabled` 属性
5. 点击可用月份 → `switchMonth(mi)` → 切换 `MONTHLY` 快照 → `switchTab(currentTab)` 重新渲染
6. 超出演示数据范围（非2026年1~6月）→ `alert('演示数据仅覆盖 2026 年 1~6 月')`
7. 点击面板外部 → 关闭（document click 事件）
8. 面板 `z-index: 50`

### 2.4 project-dashboard.html — Topbar 顶栏

| 组件 | DOM ID | 交互行为 |
|------|--------|---------|
| Logo | 六边形 `C` | 静态 |
| 标题 | "CITIBOT · 无人环卫运营中心" | 静态 |
| **日期选择器** | `#datePick` | 详情见 2.5 |
| 系统时间 | `#clock` | `HH:MM:SS` 格式，`setInterval(1000)` 实时刷新 |
| 呼吸灯 | `.status-dot` | 绿色圆点，CSS animation 脉冲效果（`pulse 2s infinite`），仅装饰，表示系统在线 |

### 2.5 project-dashboard.html — 日期选择器

**组件结构：**
```
[今日] [◀] 2026年06月16日 [▶] [累计]
  └─ 日历弹出面板 .cal-pop (绝对定位, z-index:100)
     ├─ 月份头部：[◀◀] 2026年 06月 [▶▶]
     ├─ 星期头：日 一 二 三 四 五 六
     └─ 日期网格：6行×7列
```

**交互规则：**
1. **"今日"按钮**（`#todayLink`）：span 元素，点击跳回当天。当 `selectedDate === today` 时颜色为默认文字色，非今日时蓝色高亮
2. **← → 箭头**：`shiftDate(-1/+1)`，不能超过今日（→ 在当天时置灰 `cursor: not-allowed`）
3. **日历面板**：点击日期标签或日历图标 → 展开/收起（`hidden` 切换）
4. **月份内切换**：日历面板内的 ◀◀/▶▶ 切换月份，上不封顶，下不超过当前月份
5. **日期格子状态**：
   - 当天：蓝色填充圆 + 白色数字
   - 选中日期：蓝色边框圆
   - 未来日期：浅灰色（`#CBD5E1`），`pointer-events: none`, `cursor: not-allowed`
   - hover：浅蓝背景
6. **"累计"按钮**：预留累计视图入口，当前灰色不可用
7. 切换日期 → `applyDim(date)` → 生成该日模拟数据 → 所有 widget 通过 `hookData()` 刷新
8. 面板 `z-index: 100`（高于 Topbar `z-index:10`），弹框内日历向上展开（`bottom: calc(100% + 8px)`）

### 2.6 统计卡片（stat-cards）

两原型共用 4 列网格布局：

```
.stat-card (bg-white, border-radius:10px, 左边框 3px 彩色)
├── .stat-card-label    — 12px, color: #64748B, 字重 500
├── .stat-card-val      — 大号数值, font-weight:700
│   └── .unit           — 13px 灰色单位，margin-left:4px
└── .stat-card-sub      — 12px 副文字（环比/说明/变化方向）
```

**颜色编码（左边框 c0~c3）：**
- c0 = `#0369A1`（蓝）
- c1 = `#7C3AED`（紫）
- c2 = `#B45309`（琥珀）
- c3 = `#047857`（绿）

### 2.7 分析能力浮窗（Insight Popover）

- **触发**：每个面板标题右侧 `?` 按钮（24×24px 圆形，`.insight-btn`）
- **内容源**：`INSIGHTS` 对象（key → {title, text}），共 21 条（group-analysis）
- **定位**：`position: fixed`；`z-index: 200`
- **关闭**：点击页面任意空白区域
- **内容格式**：标题（蓝色加粗 13.5px）+ 说明文字（12.5px 灰色，支持 `<ul>` 列表）

### 2.8 弹窗详情（Modal）

- **触发**：点击表格行 / 图表
- **HTML**：`#modal` 元素，默认 `hidden`
- **结构**：半透明黑遮罩（`z-index:100`）+ 白色内容卡片（`z-index:101`, max-width:960px, max-height:90vh, `overflow-y:auto`）
- **关闭**：点击遮罩 / × 按钮 → `closeModal()` → 清空 `#modalBody` + `hidden=true`

---

## 3. 数据模型

### 3.1 核心数据结构

```javascript
// —— 公司 ——
COMPANIES = ['厦门龙环', '福建龙环', '海口龙马', '乐安龙马', '乐安龙环']
COMPANY_COLORS = ['#0EA5E9','#10B981','#F59E0B','#8B5CF6','#EF4444']

// —— 车型基准 ——
USAGE_STANDARD = { SD15: 50, SD22C: 55 }       // 日均里程基准 (km)
// 百公里能耗基准（代码中隐式使用）:
// SD15 = 22 kWh/100km, SD22C = 30 kWh/100km

// —— 月度快照 per 公司 ( MONTHLY[name] ) ——
{
  mileage:    215,     // 月总里程 (km)，所有车辆合计
  duration:   35.8,    // 月运行时长 (h)
  energy:     61,      // 月总能耗 (kWh)
  water:      1280,    // 月用水量 (L)
  takeovers:  28,      // 月接管总次数
  tasks_done: 41,      // 已完成任务
  tasks_total:50,      // 计划任务
  completion: 82.0,    // 完成率 (%)
  online_rate:92.9,    // 在线率 (%)
  vehicles: [          // 车辆明细（含车型标签）
    { id:'VHC-01', model:'SD15',  mileage:58, energy:12, daily_avg:42 },
    { id:'VHC-09', model:'SD22C', mileage:72, energy:24, daily_avg:45 },
    ...
  ]
}

// —— 月度趋势 per 公司 ( MONTHLY_TREND[name] ) ——
{
  months:    ['1月','2月','3月','4月','5月','6月'],
  mileage:   [205, 210, 215, 218, 222, 228],
  energy:    [58,  60,  61,  62,  63,  65],
  takeovers: [30,  29,  28,  27,  26,  25],
  completion:[78,  80,  82,  83,  85,  86]
}
```

### 3.2 车型维度数据拆分逻辑

**核心原则：所有能按车型拆分的指标，都按车型独立计算，不混合展示。**

**百公里能耗（按车型）：**
```
车型百公里能耗 = Σ(该车型所有车辆月能耗) / Σ(该车型所有车辆月里程) × 100
```
- 过滤 `MONTHLY[name].vehicles` 中 `model === _currentModel` 的车辆
- 汇总 `mileage` 和 `energy` → `modelKwh100 = totalEnergy / totalMileage × 100`

**百公里接管率（按车型）：**
```
车型接管次数 = ceil(公司总接管次数 × 车型里程 / 公司总里程)
车型百公里接管率 = 车型接管次数 / 车型里程 × 100
```
- 接管次数无法精确按车型拆分（本批次数据中接管事件未绑定 vehicle_id），使用**里程比例估算**
- 局限性说明：假设接管概率与行驶里程成正比，实际车型之间接管率可能有系统性差异。真实数据接入后应直接按 vehicle_id 汇总

**完成率**：公司级别指标，不拆分车型。`completion = tasks_done / tasks_total × 100`

---

## 4. Tab 1：运营总览（overview）

### 4.1 页面布局

```
┌─────────────────────────────────────────────────────┐
│  stat-cards (4列)                                    │
│  [SD15清扫总里程] [单车月均] [百km能耗] [百km接管率]   │
├─────────────────────────────────────────────────────┤
│  panel: 项目公司运营排名                              │
│  表头: 公司 | 车辆 | 完成率↕ | 百km能耗↕ | 百km接管率↕ │
│  行: ★厦门龙环 | 3辆 | 82.0% | 22.5 | 12.1        │
│      ★福建龙环 | 3辆 | 90.6% | 20.8 | 4.8         │
│      ...  (仅显示有该车型的公司)                      │
│  底部说明文字: 排名逻辑...                            │
├──────────────────────┬──────────────────────────────┤
│  panel: 产出趋势      │  panel: 交付质量走势          │
│  [单折线·车型月总里程]  │  [5折线·各公司完成率(%)]     │
│  点击→弹出趋势明细弹窗  │  点击→弹出完成率明细弹窗      │
└──────────────────────┴──────────────────────────────┘
```

### 4.2 统计卡片（4 张，全按车型）

| 位置 | 标题 | 主值 | 副文字 | 数据来源 |
|:--:|------|------|--------|---------|
| c0 | `{车型} 清扫总里程` | `agg.mileage` km | `{agg.vehicles} 辆车·{agg.companies} 家公司` | `_overviewAgg[model]` |
| c1 | `{车型} 单车月均里程` | `agg.mileage / agg.vehicles` km | `↑/↓ {abs(avg-40)} km` vs 基准线 40km | 同上 |
| c2 | `{车型} 百公里能耗` | `agg.energy/agg.mileage×100` kWh | `车型基准 {USAGE_STANDARD[model]×0.44} kWh` | 同上 |
| c3 | `{车型} 百公里接管率` | `agg.takeovers/agg.mileage×100` 次 | "▼ 持续优化中"（固定文字） | 同上 |

**聚合对象** `_overviewAgg[model] = { mileage, energy, takeovers, vehicles, companies }`
- 仅统计有该车型的公司（`vehicles > 0`）才计入 `companies`

### 4.3 项目公司运营排名表

**表头列定义（5 列）：**

| 列名 | 字段 | 可排序 | 颜色编码规则 | 点击行 |
|------|------|:--:|------|:--:|
| 公司 | `name` + 8px 彩色圆点 | ✓（字母序） | 圆点 = `COMPANY_COLORS[idx]` | 弹出公司详情弹窗 |
| 车辆 | `vehicles` + "辆" | — | — | — |
| 完成率 | `completion` | ✓（默认降序） | <70 红/70-80 橙/≥95 绿 | — |
| 百km能耗 | `kwh100` | ✓ | 偏离>+10%红/>+5%橙/<-5%绿 | — |
| 百km接管率 | `takeoverRate` | ✓ | 偏离>+50%红/>+20%橙/<-30%绿 | — |

**排序状态变量：** `_overviewSortKey` + `_overviewSortDir`（1=升/-1=降）
**null 值处理：** 无该车型的公司排在末尾

**颜色编码基准（动态计算）：**
```javascript
// 集团均值 = 当前选中车型的所有公司加权平均
grpKwh100 = Σ(各公司车型energy) / Σ(各公司车型mileage) × 100
grpTakeoverRate = Σ(各公司车型takeovers) / Σ(各公司车型mileage) × 100
// 偏离度 = (公司值 - 集团均值) / 集团均值
```

**交互：点击行 → `openOverviewCompanyDetail(company)` → 弹窗**

**弹窗内容（5 个区域）：**
1. **4 张卡片**：作业完成率、百公里能耗、百公里接管率、车辆数＋月里程
2. **左图 `modalOvdTrend`**：柱状（里程 km）+ 折线（能耗 kWh）双 Y 轴，6 个月趋势
3. **右图 `modalOvdKwh`**：折线（完成率 %，绿色面积）+ 折线（百 km 能耗，黄色菱形标记）双 Y 轴
4. **数据明细表**：逐月 5 行指标（月里程/能耗/百 km 能耗/接管次数/完成率）
5. **车辆明细表**：列 = 车辆编号 / 车型 / 月里程 / 日均里程 / 利用率标签 / 百 km 能耗

**利用率标签：** `daily_avg >= USAGE_STANDARD[model]` → 绿色 "达标" / 橙色 "未达标"

### 4.4 产出趋势图（overviewTrendChart）

| 配置 | 值 |
|------|-----|
| 图表类型 | 单折线，smooth:true |
| X 轴 | 1月~6月，category |
| Y 轴 | 里程(km)，name:'km', nameGap:8 |
| 数据 | 当前车型月度总里程 = `Σ(MONTHLY_TREND[name].mileage[i] × 车型里程/公司总里程)` |
| 颜色 | `#0EA5E9` |
| 点标记 | 小圆点 size:4 |
| 交互 | 点击图表 → `openOverviewTrendDetail(monthlyTotal)` → 弹窗（大柱状图+5条公司线+逐月数据表） |
| grid | `{ left:60, right:16, top:20, bottom:32 }` |

### 4.5 交付质量走势图（overviewCompletionChart）

| 配置 | 值 |
|------|-----|
| 图表类型 | 多折线，smooth:true |
| X 轴 | 1月~6月 |
| Y 轴 | 完成率(%)，min:0 max:100，name:'%', nameGap:8 |
| 数据 | 5 家公司各 1 条线，`MONTHLY_TREND[name].completion` |
| 颜色 | 各公司对应色 |
| 点标记 | 圆点 size:4 |
| 交互 | 点击图表 → `openOverviewCompletionDetail()` → 弹窗（5条线+虚线集团均值+热力图） |
| grid | `{ left:60, right:16, top:30, bottom:32 }` |

### 4.6 内联展开行

点击排名表中的公司行 → 行下方插入 `tr.expand-row`：
- 3 个迷你 ECharts（height:120px）：完成率趋势 / 单车月均里程 / 百公里能耗
- 再次点击 → 折叠（remove 元素）
- `_expandedCompany` 追踪当前展开的公司名
- 展开另一个公司时，先移除已存在的展开行

---

*（Part 2 继续：Tab 2 效率与能耗 + Tab 3 安全与接管 + Tab 4 质量与价值）*
