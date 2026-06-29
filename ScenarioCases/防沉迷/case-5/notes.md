# case-5 抖音防沉迷卡片 · 布局与校验记录

## 需求拆解
- 顶部：应用使用时长监控标题
- 中部：横向进度条，正常部分 + 超出部分双色区分（橙色 = 超出时长）
- 中部参数：今日总时长 + 超出多少
- 底部：按钮 → 跳系统设置改每日使用上限

## 取舍（L2）
- 主答案：今日已超出每日使用上限。
- 支撑事实（≤2）：今日总时长 95 分钟、已超出 +35 分钟。
- 主动作（1）：设置使用上限（跳系统设置）。
- 进度条仅作可视化承载，两端用极弱 10fp 刻度（上限/今日）锚定语义，不复述与下方相同单位的重复数值；下方 stats 才是事实主承载。

## 尺寸 / 布局（2x2，140x140，padding 12 → 内容 116x116）
- 纵向 4 段 Column，itemMargin 10，spaceBetween：
  - title 18vp（16fp）
  - barWrap：bar 12 + 6 + 刻度 12 ≈ 30vp
  - stats 约 30vp（10fp 标签 + 14fp 值）
  - cta 30vp 按钮（热区 > 24vp）
  - 段高合计 ≈ 108 + 3×10 itemMargin 余量由 spaceBetween 吸收，底部按钮贴近安全区底边。
- 双色进度条：固定 116vp 轨道 Row（clip、borderRadius 6），内放两个固定宽度 Stack 段：
  - 正常段 73vp（蓝），超出段 43vp（橙），合计 116vp = 轨道宽，无伸缩依赖。
  - 比例锚定 60 上限 / 95 实际：60/95≈0.63 → 73vp；超出 35/95≈0.37 → 43vp。
- stats 用 spaceBetween 左右两列，左列 start、右列 end 对齐，宽度由内容估算，116vp 内充足。

## 颜色来源（color-token-system）
- root 背景 background_primary `#ffffffff`
- 标题/总时长值 font_primary `#e5000000`；标签 font_tertiary `#66000000`
- 进度正常段 / 按钮填充 brand `#ff0a59f7`，按钮文字 font_on_primary `#ffffffff`
- 超出段 / 超出值 alert `#ffed6f21`（真实“超出阈值”状态语义，单一状态色，未做整卡主题）
- 轨道底 comp_background_secondary `#19000000`
- 单一主场景色族（brand 蓝）+ 1 个状态色（alert 橙），符合克制边界。

## 事件（click-event manifest）
- manifest 仅提供 clickToDeeplink → 设置应用 `intelligent_scene_entry`（情景/专注模式，系统设置内数字健康相关页面），为可用的最接近合法目标。
- bundleName/abilityName/uri 使用 supportedTargets 中固定组合，未编造参数。

## 模型内置校验
- L0：三行 JSONL（createSurface/updateComponents/updateDataModel），v0.9 + extended.catalog，2x2 尺寸 surface=root=cardspec 一致，root 承载 width/height/padding/borderRadius/clip/backgroundColor；仅用 Form 允许组件。
- 绑定：所有表达式路径（app.title、usage.todayLabel/limitLabel/overLabel）均在 updateDataModel.value 存在。
- 布局：Row/Column 宽高预算成立，进度两段固定宽度合计=轨道宽，按钮显式 width/height，无默认伸缩依赖关键布局。
- 颜色：均可回溯 token，DSL 仅输出 hex。
- 受保护文本：标题、CTA、主指标均 maxLines 1 + textOverflow none，宽度充足不截断。
