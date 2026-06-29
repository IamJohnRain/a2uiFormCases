# 睡眠监测卡片 case-5 设计笔记

## 取舍
- 主答案：睡眠时长 6小时45分（hero 数值）。
- 支撑事实：状态「睡眠不足」（顶部）+ 圆环可视化（同一事实的进度可视化，不复述数值）。
- 主动作：设定早睡提醒（底部按钮，带闹钟图标）。
- 每个组件信息职责互斥：状态/主数值/进度可视化/动作各一。

## 尺寸与布局（2x2）
- root 140x140，padding 12，内容区 116x116，borderRadius 18，clip。
- Column 三段（status-hero-action），spaceBetween + itemMargin 6。
  - status 行 16vp（小圆点 + 「睡眠不足」12fp）
  - hero Stack 64x64（ring 64 + 居中数值 14fp + 「睡眠时长」10fp）
  - action 30vp（Stack：磨砂胶囊视觉 Row[图标16+文案14] + 透明 Button 点击层）
- 纵向预算：16 + 64 + 30 + itemMargin*2(12) = 122，spaceBetween 在 116 内分布，底部贴近安全区。
- 动作区 116 宽与内容同边界，底边距约 12vp。

## 颜色来源（scene: sleep.night）
- baseSources：multi_color_06 (#ac49f5)、multi_color_aux_06 (#c386f0)、background_tertiary(dark) #202224。
- root 渐变 temporal-band 夜间深紫：#241046 → #3A1C6B → #2A1656（基于深紫夜色族派生，低对比氛围带）。
- 强调色 ring/dot = #C386F0（multi_color_aux_06 light），前景文字用 on-color 白色系（#FFFFFFFF / #E5FFFFFF / #99FFFFFF）。
- 磨砂动作背板 #33FFFFFF（中性半透明白，等价 comp_background_secondary 思路）。
- 单一主场景色族，无第二高饱和前景。

## 事件
- 按钮 onClick = clickToDeeplink → 闹钟 App（com.huawei.hmos.clock / com.huawei.hmos.clock.phone）。来自 click-event manifest 合法目标。

## 素材
- 闹钟图标：resource/sleep_widget/icon_alarm_clock1.png（睡眠监督场景匹配），16x16 contain。

## 校验
- L0：三行 JSONL、v0.9、extended catalog、2x2 尺寸一致、root shell 完整、仅 Form 组件、按钮用 onClick。
- L1：纵向预算成立，受保护文本（状态/主数值/CTA）单行完整，按钮热区 116x30 >24vp。
- L2：信息职责互斥，进度环不复述数值，颜色可回溯，静态卡 CardSpec 仅 suggestSize。
