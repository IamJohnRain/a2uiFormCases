# 深圳天气卡片 case-2 设计说明

## 尺寸与场景
- 尺寸：2x2（140x140，root borderRadius 18，clip true，padding 12，内容区 116x116）。
- 场景：weather.morning.cool，深圳天气一览。单一主答案=当前温度。

## 信息职责（互斥，无重复）
- 城市（对象）：城市名 `深圳`。
- 天气图标（状态识别，非文字重复）：care_widget/icon_weather1.png。
- 主数值（hero）：当前温度 `28°`，40fp 单一绝对主数值。
- 支撑事实 1：天气状况 `多云`。
- 支撑事实 2：当日温区 `24° / 31°`（高低温，与当前温度不重复，承载不同判断）。
- 不含动作区（静态信息卡），故 cardspec 不写 dataBindings/refreshPlan。

## 布局预算（top-hero-bottom）
- topRow：116x22，city(左) + icon 22x22(右)，spaceBetween。
- temp：116x44，40fp 主数值。
- bottomRow：116x20，cond(左) + range(右)，spaceBetween，itemMargin 8。
- 纵向：22 + 6 + 44 + 6 + 20 = 98 <= 116 内容高，底部留白约 18vp 合理（spaceBetween 自然贴底）。
- 受保护文本（城市/温度/状况/温区）均 maxLines:1 + textOverflow:none，宽度充足无截断。
  - city `深圳` ≈ 2*16=32vp；icon 22 + gap 6 + 32 = 60 <= 116。
  - temp `28°` ≈ 3字*~24=72 < 116。
  - cond `多云` ≈ 28vp；range `24° / 31°` 数字英文约 0.6*12≈7.2/字符，约 60vp；28+8+60=96 <= 116。

## 颜色来源
- root 渐变：scenePalette weather.morning.cool；baseSources=background_primary(#ffffff)/background_secondary(#fff1f3f5) 派生低饱和冷蓝表面；derivedRoles rootStart=#FFEAF2FD, rootEnd=#FFD2E4F7。线性渐变 2 stop，低饱和高明度。
- 文本：font_primary light `#e5000000`（city/temp）、font_secondary `#99000000`（condition）、font_tertiary `#66000000`（range）。浅色背景上使用 font_primary/secondary/tertiary，配对合规。

## 校验
- L0：genui 三行 JSONL（createSurface/updateComponents/updateDataModel），version v0.9，catalog extended，2x2 尺寸 DSL 与 cardspec 一致；root 引用存在组件，含 width/height/padding/borderRadius/clip/表面样式。
- 绑定：所有表达式 `$__dataModel.weather.*` 均在 updateDataModel.value 中存在。
- 仅用 Form 支持组件（Column/Row/Text/Image）；图片为素材库本地路径，无网络图/SVG/emoji。
- 无事件能力调用，符合静态卡片定位。
