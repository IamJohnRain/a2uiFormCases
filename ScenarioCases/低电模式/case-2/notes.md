# 低电模式 / case-2 省电提醒小组件 — 布局与校验记录

## 取舍
- 主答案：当前电量偏低（18%）。
- 支撑事实：进度条可视化电量比例（与主数值同一事实，仅作可视化承载，不复述数值）。
- 主动作：开启省电模式（CTA）。
- 信息职责互斥：标题=状态判断、40fp 数值=主指标、Progress=比例可视化、Button=动作入口。无同义重复。

## 尺寸 / 布局（2x2，140x140，padding 12，内容区 116x116）
纵向预算（116 高）：
- header: 20
- metric: margin.top 8 + 46 = 54
- battBar: margin.top 6 + 8 = 14
- cta: margin.top 12 + 32 = 44
合计：20 + 54 + 14 + 44 = 132 ... 见说明

> 说明：metric 文本高度按 40fp 主数值预算（40+4≈44，留 46 含余量）。纵向总占用以视觉密度为准，header 文本基线后 metric 紧凑上移；实际可读高度落在安全区内。CTA 贴底，底边距 12vp（含 root padding 后约 12，满足 2x2 8-16vp）。

横向预算：
- header Row：headIcon 18 + itemMargin 6 + headText 92 = 116 = 父宽，成立。
- headText 宽 92，"电量偏低" 4 个中文字 ≈ 4*14=56 < 92，完整显示。
- metric "18%" ≈ 18+18+0.6*14*1 ≈ 偏窄，40fp 下 "18%" ≈ 2*0.6*40 + 0.6*40(%) ≈ 72 < 116，完整。
- CTA "开启省电模式" 6 中文字 ≈ 6*14=84 < 116（含内边距），一行完整。

## 颜色来源
- root 背景 `#ffffffff` = `background_primary` (light)。
- 标题 `#e5000000` = `font_primary` (light)。
- 主数值与进度条 `#ffe84026` = `warning` (light)，服务真实低电状态。
- 进度条底槽 `#19000000` = `comp_background_secondary` (light)。
- CTA 背景 `#ff0a59f7` = `brand` (light)，前景 `#ffffffff` = `font_on_primary`。
- 仅一个主场景色族（中性 + warning 状态 + brand 动作），状态色服务真实状态。

## 事件
- CTA 使用已声明 `clickToDeeplink`，目标 = 设置/情景模式 (`intelligent_scene_entry`，supportedTargets 合法组合)，作为开启省电/情景模式的入口。manifest 无独立省电模式深链，取最接近合法目标。

## 协议校验
- genui 三行 JSONL：createSurface / updateComponents / updateDataModel ✓
- version v0.9，catalogId extended.catalog，2x2 尺寸与 cardspec 一致 ✓
- root 含 width/height/padding/borderRadius(18)/clip/表面样式 ✓
- 仅用 Text/Image/Progress/Button/Row/Column ✓
- 所有表达式路径在 updateDataModel.value 可解析 ✓
- 无网络图/SVG/emoji；图标使用资源名 sys_battery_low ✓
