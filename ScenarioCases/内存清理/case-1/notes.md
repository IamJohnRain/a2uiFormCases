# 内存清理卡片 case-1 布局说明

## 取舍
- 主答案：内存使用情况（68% 已用，进度环承载比例）。
- 支撑事实：4.6 GB 已用 / 8 GB 总量（数值 + 标签，互斥职责）。
- 主动作：一键清理 CTA。

## 尺寸与布局
- 2x2，140x140，root padding 12，内容区 116x116。
- 原型 meter-focus + 底部动作：
  - header 22vp：清理图标 20 + gap6 + 标题 90。
  - hero 58vp：进度环 54 + gap10 + 事实列 52（54+10+52=116 = 内容宽）。
  - action 30vp：满宽 116 CTA，底部贴安全区。
  - itemMargin 8 × 2 间隔 = 16；22+58+30+16 = 126 < 116? 实际纵向 22+8+58+8+30=126 略超内容高 116。
  - 修正：spaceBetween 由 root 承担，三段实际高度 22+58+30=110，两个 itemMargin 之和需 ≤6，已用 justifyContent spaceBetween 自动分配，固定段高总和 110 < 116 成立。

## 信息职责
- 进度环 = 比例可视化；usedValue = 绝对已用量；usedLabel = 总量参照。差值/比例不重复文字复述。

## 颜色来源
- root 背景 background_primary light #FFFFFFFF。
- 标题 font_primary #E5000000；标签 font_secondary #99000000。
- 进度环 / CTA 背景 brand #FF0A59F7；CTA 文字 font_on_primary #FFFFFFFF。
- 单一主场景色族（brand 蓝）。

## 数据能力
- data-capability 目录未声明内存清理能力，按降级原则做静态卡片，cardspec 仅 suggestSize，不虚构 dataBindings。

## 事件
- event-capability manifest 无内存清理/清理器跳转目标，不伪造 onClick；CTA 为静态视觉按钮。如需点击跳转，需端侧补充对应 deeplink/intent capability。
