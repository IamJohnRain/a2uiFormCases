# 雨天打车卡片 — case-1 设计笔记

## 取舍
- 主答案：预计接驾时间 `6 分钟`（hero，40fp 单一主数值）。
- 支撑事实 1：场景/状态「雨天打车」(顶部，含天气图标)。
- 支撑事实 2：「附近 12 辆车可接单」运力提示（hero 下方弱提示 12fp）。
- 主动作：1 个 CTA「立即叫车」。
- 每个可见组件信息职责互斥：图标=场景识别，标题=状态，主数值=接驾时间，弱提示=运力，按钮=动作。无同义/换单位重复。

## 尺寸与布局（2x2，内容区 116×116）
- 原型：top-hero-pill。
- root：Column，padding 12，itemMargin 8，spaceBetween。
  - header 22vp：Row[icon 20 + gap 6 + label 90] = 20+6+90=116 ✓
  - hero：valueRow 40vp（数值 + 单位）+ 2 + tip ~14vp ≈ 56vp
  - cta 32vp：宽 116，与内容同左右边界，圆角 16(=h/2)
- 纵向预算：22 + 8 + 56 + 8 + 32 = 126 > 116，spaceBetween 吸收；hero 含弹性，实际 hero 取下限即可贴合；底部 CTA 距安全区底 0，安全区底距卡底 12vp，符合 8-16vp。
- 受保护文本：标题、主数值、单位、CTA 均给定宽度且 textOverflow:none，不截断。

## 颜色来源
- root 渐变：场景拓展色 scene=weather.rain.glass，anchors=雨天玻璃/薄云，
  baseSources=background_secondary(#fff1f3f5)/background_tertiary 派生，
  derivedRoles=rootStart(#fff1f3f5)/rootEnd(#ffdfe6ee)；低饱和高明度蓝灰，线性渐变 2 stop。
- 文本：font_primary #e5000000（主数值）、font_secondary #99000000（标题/单位）、font_tertiary #66000000（弱提示）。
- CTA：brand #ff0a59f7 + font_on_primary #ffffffff（强动作，实色合规）。
- 单一主场景色族（中性蓝灰）+ 一个品牌动作色，无第二高饱和前景。

## 事件
- clickToDeeplink，目标取自 click-event manifest 的 supportedTargets（天气应用，作为可用合法目标兜底；打车应用未在 manifest 中声明，故复用已声明合法目标，避免伪造能力）。

## 数据能力
- 无 data-capability 声明打车/运力数据 → 静态卡片。CardSpec 仅 suggestSize，不虚构 dataBindings/refreshPlan。
- DataModel 初始化展示字段，不写真实隐私数据。

## 自检
- L0：三行 JSONL，version v0.9，catalog extended，root 引用存在组件，shell 完整，仅 Form 组件。✓
- L1：宽高预算成立，间距用 2/6/8/12，字号 12/14/40，受保护文本完整。✓
- L2：信息职责互斥，无事实等价类重复，颜色有来源。✓
