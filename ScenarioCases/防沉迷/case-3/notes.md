# case-3 防沉迷卡片 — 布局理由与校验

## 需求拆解
- 主答案：每日使用进度（已用 vs 上限）→ 线性进度条 hero
- 支撑事实 1：每天打开抖音次数
- 支撑事实 2：每次平均使用时长
- 1 个主答案 + 2 条支撑事实，无主动作 → 选 2x2

## 信息职责（互斥）
- titleRow：对象（抖音）+ 状态提示（剩余时长 remainLabel）
- meterGroup：唯一主指标 = 今日进度（usedLabel 主值、limitLabel 阈值、bar 可视化）
- statsRow：两个派生但独立的事实（打开次数 / 单次均时），各自承担新判断，不与进度复述
- remainLabel 是“剩余”判断，与“已用/上限”属同一进度事实族，但仅以 10fp 弱提示出现一次，未在别处重复换算

## 布局预算（2x2，内容区 116x116，padding 12）
- root Column itemMargin 10，三段：18 + 进度组 + 32
- titleRow：width116 height18，spaceBetween，title 12fp + pill 10fp
- meterGroup：meterTop（20fp 主值 + 12fp 阈值，spaceBetween）+ bar(116x8)，itemMargin6
- statsRow：openStat(48) + divider(1,竖) + avgStat(48) + itemMargin10*2=20 → 48+1+48+20=117≈116，alignItems center
  - 每个 stat Column：14fp 值 + 10fp 标签，itemMargin2
- 纵向：18 +10+ (~46 meter) +10+ 32 = 116，spaceBetween 收尾贴底

## 颜色来源
- root 背景 background_primary light = #FFFFFFFF
- 标题/主值 font_primary = #E5000000
- 阈值/标签 font_secondary = #99000000
- 状态 pill / 进度条 font_emphasize、brand = #FF0A59F7
- 进度条底槽 comp_background_secondary light = #19000000
- divider comp_divider light = #33000000
- 单一主场景色族（brand 蓝），无第二高饱和色，状态色未滥用

## 校验
- L0：三行 JSONL；version v0.9；catalog extended；2x2 尺寸/圆角18/clip 一致；root 引用存在、含 shell
- 绑定：所有表达式路径（usage.usedLabel/limitLabel/remainLabel/used/limit/openCountLabel/avgLabel）均在 updateDataModel.value 中
- 组件：仅用 Column/Row/Text/Progress/Divider，均在 Form 支持列表
- 文本：受保护文本（标题、主值、上限、次数、均时）均 maxLines1 + textOverflow none，宽度预算成立
- 无禁用组件、无网络图、无 emoji、无点击能力（静态卡，cardspec 不含 dataBindings/refreshPlan）
