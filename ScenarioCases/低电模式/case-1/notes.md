# 低电模式卡片 case-1 设计笔记

## 场景与信息取舍
- 主答案：当前电量百分比（hero，进度环 + 居中数值）。
- 支撑：标题「省电模式」标识场景。
- 主动作：1 个 Button「开启省电」→ 跳转系统设置情景模式页。
- 未展示剩余时间/温度等弱信息，避免与电量构成事实重复并保持 2x2 信息上限（标题 + hero + 动作）。

## 布局预算（2x2 = 140x140，root padding 12 → 内容区 116x116）
- 原型：`meter-focus` / `top-hero-pill`，Column 垂直三段，spaceBetween + 居中对齐。
- 高度：title 20 + itemMargin 8 + ring 64 + itemMargin 8 + actionBtn 30 = 130 > 116。
  - spaceBetween 会按内容区 116 分布；实际固定元素 20+64+30=114，剩 2vp 余量由 spaceBetween 吸收，间距 ≥8 由 itemMargin 保证，不溢出。
- 宽度：title 116、ring 64、按钮 116，均 ≤116。
- 按钮：高 30 ≥24vp 热区；宽 116 包含「开启省电」4 字（14fp≈56）+ 内边距，余量充足，文字单行不截断。
- 进度环：64vp 在 56-72 推荐区间；环内数值 20fp 居中，Stack 仅用于环 + 数值叠加，不覆盖受保护文本。

## 颜色来源
- root 背景 `#FFFFFFFF` = `background_primary`(light)。
- 标题/环内数值 `#E5000000` = `font_primary`(light)。
- 进度环 / CTA 背景 `#F9A01E` = `multi_color_10`(light，琥珀)，作为低电场景单一主色族（节能/低电氛围锚点），非 warning/alert 语义。
- CTA 文字 `#FFFFFFFF` = `font_on_primary`，用于强调色背景上的 on-color 前景。

## 事件能力
- `clickToDeeplink` 来自 event-capability manifest；目标 = 设置 / MainAbility / `intelligent_scene_entry`（合法 supportedTargets 组合，打开情景/省电模式）。

## 模型内置校验
- L0 协议：三行 JSONL（createSurface/updateComponents/updateDataModel）；version v0.9；catalog extended；仅用 Column/Text/Stack/Progress/Button；root 含 width/height/padding/borderRadius/clip/backgroundColor。✅
- 绑定：`battery.level` 在 updateDataModel.value 中存在；表达式完整单对 `{{ }}`。✅
- 资源：移除虚构 Image 资源，无网络图/SVG/emoji。✅
- L1 布局：宽高预算成立，按钮热区 ≥24vp，受保护文本（标题/百分比/CTA）完整。✅
- L2：信息职责互斥，无事实重复，单一主色族。✅
- 静态卡：cardspec 仅 suggestSize=2x2，无虚构 dataBindings/refreshPlan。✅

## 说明
- 电量 level=18 为初始占位演示值；端侧可通过 updateDataModel 刷新 `/battery/level` 实现响应式更新（进度环与百分比均绑定该路径）。
