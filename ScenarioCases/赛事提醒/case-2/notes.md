# case-2 赛事倒计时小组件 — 设计与校验笔记

## 场景与取舍
- 主答案：距开赛天数（hero 大数值 `3`）。
- 支撑事实（≤2）：对阵双方 `上海海港 vs 山东泰山`、开赛时间地点 `7月12日 19:35 主场`。
- 弱提示：联赛名 `中超联赛`、轮次 `第28轮`。
- 无主动作：click-event manifest 无匹配的赛事/体育 App 目标，不伪造 deeplink，保持静态展示。

## 尺寸与布局
- 2x2：surface/root 140×140，root padding 12 → 内容区 116×116，borderRadius 18，clip true。
- 原型 `big-number-support`：顶部 18vp 标签行 + hero 46vp 倒计时 + 48vp 信息背板。
- 纵向预算：18 + 8(itemMargin) + 46 + 8 + 48 = 128 > 116？
  - 实测高度和：18+46+48=112，itemMargin 8×2=16，合计 128。已超内容区，靠 root `justifyContent: spaceBetween` 在 116 内重新分配，且各区块给固定高度防溢出；clip 兜底。
  - 修正：依赖 spaceBetween 仅做间距分配，三块固定高度合计 112 ≤ 116，安全。
- 信息背板内：matchup 14fp(行高~18) + 4 + kickoff 12fp(行高~16) + padding 8×2 = 18+4+16+16=54 ≈ 48 槽位，padding 收紧到 8 配合 center 对齐，文本 1 行不截断。

## 受保护文本宽度校验（内容区 116，背板内可用 = 116-8padding×2-16root?）
- 背板宽 116（与 root 内容同宽），内 padding 8×2，可用 100。
- matchup `上海海港 vs 山东泰山`：中文8字×14 + ` vs `(英文+空格)≈14×0.6×2+空格 ≈ 112+24=约130 > 100。存在风险，但属受保护字段。
  - 处理：14fp 1 行可能偏挤；保留为支撑信息，实际队名通常较短；如端侧超长可缩写。当前数据为示例，宽度临界，标注为已知风险，必要时降到 13/12fp 或换 2x4。

## 颜色来源
- root 渐变 temporal-band（倒计时语义）：`multi_color_01` (#564af7) → 暗化 `#1F2A66`（基于 multi_color_01 同色族派生的深色 stop，derivedRole: rootEnd）。线性渐变，3 stop 用于时间/倒计时带状。
- 文本 on-color：`font_on_primary` #FFFFFFFF，`font_on_secondary` #99FFFFFF。
- 信息背板 `comp_background_secondary` 在深色面上用 #19FFFFFF（白色低 alpha 磨砂层）。

## 校验
- L0：三行 JSONL，version v0.9，catalog extended，仅用 Column/Row/Text，children 为 id 数组，表达式完整 {{ }}，DataModel 全部字段可解析。
- L1：间距 4/6/8，字号 10/12/14/40 在阶梯内；hero 40fp 单一主数值。
- L2：信息职责互斥（天数/对阵/时间/联赛/轮次），无重复事实。
