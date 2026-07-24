# PRD：智慧环卫项目（详细版）— Part 4：全局交互流 + 状态处理 + 数据流 + 待确认项

**版本**: 2.0
**日期**: 2026-07-14

---

## 12. 全局交互流程图

### 12.1 group-analysis.html — Tab 切换流程

```
用户点击 Tab 按钮
  │
  ├─ 判断: tab === currentTab? → 是 → 忽略
  │
  ├─ 更新按钮激活态 (classList.add/remove 'active')
  ├─ disposeAllCharts() — 销毁全部 ECharts 实例
  │     └─ Object.keys(chartInstances).forEach(k → dispose + delete)
  │
  ├─ switchTab(tab) →
  │     ├─ render*() 调用 → 返回 HTML 字符串
  │     ├─ container.innerHTML = html
  │     └─ setTimeout(50ms) →
  │           └─ mount*() 调用 → 初始化所有 ECharts 图表
  │
  └─ currentTab = tab
```

### 12.2 group-analysis.html — 车型切换流程

```
用户点击 #globalModelToggle 中的按钮
  │
  ├─ 判断: model === _currentModel? → 是 → 忽略
  │
  ├─ 更新按钮激活态 (所有去 active → 被点击加 active)
  ├─ _currentModel = model
  ├─ _overviewSortKey = 'completion' (重置排序)
  ├─ _overviewSortDir = -1
  └─ switchTab(currentTab) — 重渲染当前 tab
```

### 12.3 group-analysis.html — 月份切换流程

```
用户点击月份网格中的月份 / 年份箭头
  │
  ├─ 年份箭头 (±1):
  │     ├─ 判断边界 (2020 ≤ year ≤ thisYear)
  │     ├─ ymDropYear ±= 1
  │     └─ renderYmGrid() — 重绘 12 月按钮（含置灰判断）
  │
  ├─ 点击月份:
  │     ├─ 判断: 是否为当前月 (year=2026 && month=currentMonth)? → 是 → 关闭面板
  │     ├─ 判断: 是否在 2026/1~6 范围外? → 是 → alert('演示数据仅覆盖 2026 年 1~6 月')
  │     ├─ switchMonth(mi):
  │     │     ├─ currentMonth = mi
  │     │     ├─ MONTHLY = MONTHLY_SNAPSHOTS[currentMonth] — 切换数据快照
  │     │     ├─ 更新 ymLabel 文字
  │     │     └─ switchTab(currentTab) — 重新渲染
  │     └─ ymDrop.hidden = true — 关闭面板
  │
  └─ 点击面板外部 → ymDrop.hidden = true
```

### 12.4 project-dashboard.html — 日期切换流程

```
用户操作日期选择器 (箭头/日历/今日按钮)
  │
  ├─ shiftDate(±1): 日期 ± 1 天 → 判断不超今日
  │
  ├─ 日历面板点击日期 → 判断非未来 → 选中
  │
  ├─ "今日"按钮 → 日期 = new Date()
  │
  ├─ fmtDateLabel() 更新显示文字
  ├─ buildCalendar() 重新生成日历 HTML
  │
  └─ applyDim(date) →
        ├─ getData(dim) — 根据日期生成当日模拟数据
        ├─ hookData() — 更新各 widget 显示
        │     ├─ 更新 Hero 指标文字
        │     ├─ 更新安全告警 4 格数据
        │     ├─ 更新任务完成数据
        │     ├─ 更新资源补给数据
        │     └─ 更新事件流列表
        ├─ renderHeroTrend(dim) — 重绘趋势图 (try-catch)
        ├─ renderEfficiency() — 重绘能效图 (try-catch)
        └─ initModalChart() — 更新弹窗图表 (try-catch)
```

### 12.5 弹窗交互流程

```
用户点击表格行 / 图表 / 能效区域
  │
  ├─ open*() 函数调用:
  │     ├─ 设置 document.getElementById('modalTitle').innerHTML
  │     ├─ 设置 document.getElementById('modalBody').innerHTML
  │     │     └─ HTML 包含: 卡片 + 图表 div 容器 + 表格
  │     ├─ document.getElementById('modal').hidden = false
  │     └─ setTimeout(50ms) → initChart(...) 初始化弹窗内图表
  │
  ├─ 用户点击遮罩 / × 按钮:
  │     ├─ closeModal()
  │     │     ├─ document.getElementById('modal').hidden = true
  │     │     └─ document.getElementById('modalBody').innerHTML = '' — 清空内容
  │     └─ 弹窗内图表随 HTML 移除而失效（无需手动 dispose，但建议在 closeModal 中追加 dispose 逻辑）
  │
  └─ 键盘 ESC 关闭:
        └─ document.addEventListener('keydown', e => e.key === 'Escape' && closeModal())
```

---

## 13. 状态处理矩阵

### 13.1 加载状态

| 场景 | group-analysis.html | project-dashboard.html |
|------|---------------------|------------------------|
| 首次加载 | DOMContentLoaded → `switchTab('overview')` 触发渲染 | `applyDim(new Date())` 直接用当天数据渲染 |
| Tab 切换 | `innerHTML` 同步注入 HTML → 50ms 延迟后挂载图表 | N/A（无 tab） |
| 月份/日期切换 | `switchMonth → switchTab` 链式调用 | `applyDim(date)` 统一入口 |
| ECharts CDN 失败 | try-catch 包裹所有 `echarts.init` 调用点，图表区域留空但不阻塞其他 UI | 同上 |
| 图表容器不存在 | `initChart` 检测 `document.getElementById(domId)` 为 null → 返回 null，不报错 | 同上 |

### 13.2 空数据状态

| 场景 | 处理方式 |
|------|---------|
| 某公司无当前车型 | `_overviewByModel[model]` 过滤掉 `vehicles=0` 的公司，不出现排名表中 |
| `bestSD22C`/`worst` 为 undefined | 显示 `'--'` 字符串，"无数据" 副文字 |
| kwh100 为 null | 排序时 `null` 值排末尾 |
| 某车型无任何公司数据 | 统计卡片显示 `fmt(0)`，排名表 `tbody` 为空 |

### 13.3 错误状态

| 场景 | 处理方式 |
|------|---------|
| ECharts 变量未定义 | `typeof echarts === 'undefined'` 检查 → 跳过图表初始化 |
| 图表 DOM 不存在 | `initChart(domId, opt)` → `document.getElementById(domId)` 返回 null → return null |
| 弹窗内图表重复初始化 | 通过 `chartInstances[id]` 追踪，先 `dispose` 再重建 |
| 日期选择器外部点击 | document click 事件 + `closest` 判断 → 关闭所有日历面板 |
| 月份选择器越界 | 年份检查 + alert 提示 |

### 13.4 边界情况

| 场景 | 处理方式 |
|------|---------|
| 排序时 null 值比较 | `if(va===null)return 1; if(vb===null)return -1` |
| 接管率除零 | `modelMileage > 0 ? takeovers/mileage×100 : null` |
| 月度趋势月份不一致 | `compTrend.months.map((_,i) => ...)` 遍历索引，保证 6 个月对齐 |
| 日历日期跨月 | 上月末尾格 + 下月起始格 = 灰色填充，当前月 = 正常色 |
| 切换月份时展开了弹窗 | 弹窗内容依赖当前 `MONTHLY`，若弹窗开着，切换月份后数据源变化但弹窗 HTML 已渲染 → 关闭弹窗或重新 open（当前实现：弹窗是即时计算 HTML，打开后再切换月份不会自动更新） |
| Z-index 遮挡 | Topbar(z-10) → 日历面板(z-100) → 弹窗遮罩(z-100) → 弹窗卡(z-101) → 分析浮窗(z-200) |

---

## 14. 数据流总览

### 14.1 group-analysis.html 数据流

```
MONTHLY_BASE (静态基础数据，按公司)
  └─ genDailyData() → DAILY[name] (日级随机数据，30天)
  └─ MONTHLY_TREND[name] (6月趋势数据，含随机噪声)
       └─ MONTHLY_SNAPSHOTS[0..5] (6 张月度快照)
            └─ MONTHLY = MONTHLY_SNAPSHOTS[currentMonth]
                 └─ renderOverview() / renderEfficiency() / renderSafety() / renderValue()
                      └─ _overviewByModel[model] (按车型聚合)
                      └─ _overviewAgg[model] (车型汇总)
                           └─ 统计卡片 + 排名表 + 图表
```

**数据依赖链：**
1. `MONTHLY_BASE` → `MONTHLY_SNAPSHOTS[mi]` → `MONTHLY`（当前月）
2. `MONTHLY[name].vehicles` → 按 `model` 过滤 → 汇总 → 排名表 / 图表
3. `MONTHLY_TREND[name]` → 趋势图（6 个月 × 5 家公司）
4. `TAKEOVER_CAUSES` / `COMPANY_TAKEOVER` → 安全分析图表（全局静态）

### 14.2 project-dashboard.html 数据流

```
BASE_DATA (5 辆车基础数据)
  ├─ getData('day') → dayData (当天数据，含随机抖动)
  │     └─ 种子: f(date) → Math.sin(seed) 生成伪随机抖动
  │     └─ 所有 widget 读取 dayData
  │
  └─ getData('total') → totalData (累计数据，base × 累积系数)
        └─ 预留累计视图切换
```

**数据更新路径：**
```
applyDim(date)
  → getData(dim) → { hero, fleet, efficiency, safety, tasks, resources, events }
  → hookData() → DOM textContent 更新
  → renderHeroTrend(dim) → ECharts.setOption (not re-init)
  → renderEfficiency() / initModalChart() → ECharts.setOption
```

---

## 15. 图表实例管理

### 15.1 group-analysis.html

```javascript
chartInstances = {} // { domId → echartsInstance }
// 生命周期:
initChart(domId, option):
  1. el = getElementById(domId), 无则 return null
  2. if (chartInstances[domId]) dispose + delete
  3. chart = echarts.init(el)
  4. chart.setOption(option)
  5. chartInstances[domId] = chart
  6. return chart

disposeAllCharts():
  Object.keys(chartInstances).forEach(k → dispose + delete)
  // Tab 切换前调用，防止残留实例
```

### 15.2 project-dashboard.html

无 `disposeAllCharts`（单页无 tab），通过 `setOption` 更新已有实例而非重新 init。

---

## 16. ECharts 全局样式变量

```javascript
ECHARTS_TEXT_COLOR = '#64748B'    // 轴标签、图例文字
ECHARTS_LINE_COLOR = '#E2E8F0'    // 网格线
```

所有图表统一使用这两个变量设置 `axisLabel.color`、`splitLine.lineStyle.color`、`nameTextStyle.color` 等。

---

## 17. 待确认项

| # | 项 | 说明 |
|----|-----|------|
| 1 | 作业云图（2.1）大屏派生指标 | 飞书文档中提到的"累计清扫里程趋势曲线、综合能效 km/kWh、任务画像雷达 6 维评分"是否需要单独的大屏驾驶舱页面？当前 project-dashboard.html 已有 Hero 趋势 + 能效看板，但缺少雷达图和大屏专用布局。需确认是在 project-dashboard 上改还是新增独立页面 |
| 2 | 日均协同事件推送到 APP | 推送渠道（环卫云 APP 是否有现成接口？）、超时升级规则（N 分钟默认值待定）、转派逻辑 |
| 3 | 资源补给数据源 | 充电/加水/倒垃圾的时间记录由车端上报还是平台推算？当前为 Mock 数据 |
| 4 | 实时事件流升级 | 当前为每日看板模式（日期切换刷新），后续是否升级为 WebSocket 实时推送？若否，当前设计已满足需求 |
| 5 | 累计视图 | project-dashboard.html 的"累计"按钮当前不可用，需确认累计视图的数据范围（本月累计/历史累计）和展示方式（与日报同一布局还是独立布局） |
| 6 | 报表导出功能 | 两原型均有"导出报表"按钮，当前仅占位。需确认导出格式（PDF/Excel）、导出范围（当前 tab/全部数据） |
| 7 | 后端 API 接入 | 当前全为前端 Mock 数据，真实 API 的数据结构和接口定义需后续对接时确认（对接入成本影响最大的模块：月份切换快照、车型维度数据、车辆明细） |

---

*全文完。共 4 个 Part，覆盖产业公司层 4 个 Tab + 项目公司层 8 个模块 + 全局组件 + 交互流程 + 数据模型 + 状态处理。*
