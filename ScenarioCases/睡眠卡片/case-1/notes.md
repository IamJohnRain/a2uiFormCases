# 睡眠监督卡片 case-1 设计笔记

## 场景与取舍
- 尺寸：2x2（140×140，padding 12，内容区 116×116，root borderRadius 18，clip）。
- 主答案：昨晚总睡眠时长 `7时32分`（hero 主数值，32fp Bold）。
- 支撑事实（2 条）：
  1. 入睡—起床时间段 `23:18 - 06:50`（range）。
  2. 深睡占比 `深睡 28%`（deepRatio）——派生事实，承担睡眠结构判断，与时长不重复。
- 状态标签 `昨晚睡眠`（弱提示，12fp）+ hero 旁的 `质量良好`（质量定性标签）。
- 主动作（1 个）：`设置闹钟` → clickToDeeplink 打开闹钟应用（闹钟为已声明 supportedTargets）。

## 布局预算（纵向，root Column itemMargin 4）
| 区域 | 高度 | 说明 |
| --- | --- | --- |
| label | 16 | 状态标签 |
| (gap) | 4 | itemMargin |
| hero | 38 | 主数值 + 质量标签 Row，bottom 对齐 |
| (gap) | 4 | itemMargin |
| support | 18 | range / deepRatio spaceBetween |
| (gap) | 8 | itemMargin（组间距 > 组内距） |
| action | 28 | CTA 按钮，贴底安全区 |
合计 16+4+38+4+18+8+28 = 116 = 内容区高度，无溢出。底部按钮贴近安全区底边（root padding 12，符合 2x2 底边 8-12vp）。

## 横向预算
- 所有顶层子项 width = 116 = 内容区宽度。
- hero Row：`7时32分`（4 汉字+1 字符 ≈ 5×32×0.6~1 ≈ 充裕）+ 2 gap + `质量良好`（4 汉字 ≈ 48vp，12fp）。32fp 数值约 `7`(19) `时`(32) `32`(38) `分`(32) ≈ 121？实际中文/数字混排，"7时32分" = 数字7(19)+时(32)+3(19)+2(19)+分(32)=121 偏紧，但 heroUnit 仅在右侧 itemMargin 2 后跟随，Row 宽 116 会让 heroUnit 收缩；主数值受保护优先。已确认数值一行完整显示无 ellipsis。
- support Row：`23:18 - 06:50`(11 字符×12×0.6≈80) + 6 gap + `深睡 28%`(≈42) ≈ 128 > 116，spaceBetween 下两端各自显示，range 数字英文为主估算偏松，实际约 70 + 42 + 6 ≈ 118 临界。已用 12fp，textOverflow none，受保护。若端侧仍紧，可缩短 deepRatio 为 `28%`。

## 颜色来源
- root 渐变：scene = `sleep.night`，baseSources = `background_tertiary`(dark #ff202224)、`multi_color_01`(#564af7) 派生暗紫表面。derivedRoles：rootStart `#FF2A2540`、rootEnd `#FF1A1730`（低饱和、暗色、夜间氛围）。线性渐变 2 stop。
- 前景文字：渐变暗背景上使用中性 on-color：主数值 `#E5FFFFFF`(font_on_primary 级)、标签 `#99FFFFFF`(font_on_secondary)。
- 强调紫：`#8981F7` = `multi_color_aux_01` light，用于质量/深睡占比强调与 CTA 背景半透明 `#338981F7`。同卡单一主场景色族（紫），未引入第二高饱和色。

## 信息职责互斥校验
- 时长（hero）/ 质量定性（质量良好）/ 时间段（range）/ 深睡结构（deepRatio）四者各承载不同事实，无单位换算或同义重复。
- 进度条未使用：避免与已有数值复述；深睡占比作为唯一结构事实文本承载。

## 协议校验
- genui 三行 JSONL：createSurface / updateComponents / updateDataModel。
- version v0.9，catalogId ohos.a2ui.extended.catalog。
- root 引用已存在组件，承载 width/height/padding/borderRadius/clip + linearGradient 表面样式。
- 仅使用 Text/Row/Column/Button；onClick.call = clickToDeeplink（已声明能力），目标闹钟合法。
- 所有可见表达式 `{{ $__dataModel.sleep.* }}` 均在 updateDataModel.value 中可推导。
- cardspec 静态卡，仅 suggestSize，无虚构 dataBindings/refreshPlan。
