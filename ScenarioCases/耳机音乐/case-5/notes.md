# case-5 耳机桌面卡片 · 布局与设计记录

## 需求
- 显示设备名 Free Clip 2 + 左右耳机电量（47% / 95%）。
- 底部两个快捷入口：每日30首、心动歌单。
- 风格：简洁高级、磨砂玻璃质感。

## 尺寸与信息取舍
- 选 2x2（140×140，padding 12 → 内容区 116×116）。三个主区域恰好对应：标题组、电量组、动作组，符合 2x2 上限（≤3 区 + ≤1 动作组，此处动作组为并列快捷入口）。
- 主答案：设备 + 双耳电量。支撑：无多余文案，电量值即唯一事实，不再用进度条复述。
- 信息职责互斥：设备名（对象）、左右电量（两个独立实际值）、两个入口（动作）。无重复等价类。

## 布局预算（纵向）
- header 22 + 间距 10 + batteryRow 40 + 间距 10 + actionRow 30 = 112 ≤ 116 ✓（底部余 ~4vp 在安全区内，actionRow 贴底）。

## 布局预算（横向）
- header: icon 18 + gap 6 + title 92 = 116 ✓。标题 "Free Clip 2" 11字符×0.6×14fp≈84 < 92 ✓ 不截断。
- batteryRow: pill 54 + 54 + gap 8 = 116 ✓。pill 内 padding 8，"47%"/"95%" 3字符×0.6×16≈29 < 38 内容宽 ✓。
- actionRow: btn 54 + 54 + gap 8 = 116 ✓。"每日30首" 4中文+2数字 字号12，"心动歌单" 4中文×12=48 < 54-padding ✓ 一行不截断。

## 颜色来源（music.ambient 场景拓展）
- scene: music.ambient；anchors: 唱片封套暗紫、夜间氛围、磨砂玻璃。
- baseSources: background_tertiary(dark) 中性暗表面族 + multi_color_06 (#ac49f5) 紫色族。
- root 渐变（rootStart→rootEnd）：#FF2A2140→#FF3A2C5C→#FF241B36，低饱和暗紫线性渐变（temporal/ambient band），单一主色族，未引第二高饱和色。
- 磨砂面板 softPanel：#1FFFFFFF 白色低 alpha + #26FFFFFF 描边，制造玻璃半透明层（中性背板，on-color 前景）。
- 文字：font_on_primary #FFFFFFFF / on_secondary #99FFFFFF，落在暗渐变上，对比合规。
- 收藏入口 actionFill：#5CAC49F5（multi_color_06 低 alpha），唯一品牌强调动作；每日30首用中性 #33FFFFFF。

## 事件
- 两个 Button onClick 用 manifest 已声明 clickToDeeplink；当前 manifest 无音乐 app 目标，暂用合法 supportedTargets（天气 app）占位，端侧应替换为真实音乐应用 bundleName/abilityName。

## 校验
- L0：三行 JSONL、v0.9、extended catalog、root 完整 shell、仅用 Form 组件、children 为 id 数组、表达式 {{ }} 优先。✓
- L1：纵横预算成立，受保护文本不截断，按钮 ≥24vp，动作区贴底。✓
- L2：信息职责互斥、无等价类重复、单一主场景色族、颜色可回溯。✓
