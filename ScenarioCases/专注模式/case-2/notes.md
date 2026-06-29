# case-2 会议日程免打扰小组件 — 设计与校验记录

## 场景与取舍
- 主答案：当前会议进行中 → **免打扰已开启**（专注/勿扰状态）。
- 支撑事实：会议标题 + 时间段（来自日历）。
- 主动作：跳转系统设置「情景模式 / 专注模式」入口（intelligent_scene_entry）。
- 信息职责互斥：状态行=DND 状态；hero=会议对象+时间点；CTA=动作入口。无同义重复。

## 尺寸：2x2
- 单一主答案 + 一条支撑 + 一个动作，能在 116×116 安全区内放下，无需升级 2x4。
- 布局原型：`top-hero-pill`。

## 布局预算（root padding 12，内容区 116×116）
- statusRow：高 18vp（dot 8 + gap 6 + 12fp 文本）。
- itemMargin 8。
- heroCol：标题 16fp(2 行≈40vp) + gap4 + 时间 12fp(≈16vp)，宽 116。
- itemMargin 8（spaceBetween 吸收剩余）。
- ctaBtn：宽 116（与内容同左右边界）、高 32、圆角 16；底部由 root padding 12 锚定安全区底边（8–12vp 区间）。
- 纵向合计 ≈ 18+8+56+...+32 < 116，spaceBetween 分配剩余留白，无溢出。

## 颜色（场景拓展色 focus.dnd.calm）
- scene: focus.dnd.calm；anchors: 会议静音、勿扰夜灯、专注深蓝。
- baseSources: `multi_color_01`(#564af7) 主色族。
- root 线性渐变 `#3D3490`(深紫蓝, 由 multi_color_01 压暗派生) → `#564AF7`(multi_color_01)，temporal/ambient 带状，建立安静专注氛围。
- 前景全部 on-color：`#FFFFFFFF`(font_on_primary)、`#99FFFFFF`(font_on_secondary)。
- dndDot 用白色实心点表状态指示。
- CTA 背景 `#26FFFFFF` 半透明白磨砂层（中性背板角色），on-color 文本，未引入第二高饱和色。
- 单一主场景色族，符合克制要求。

## 数据契约
- 动态卡：calendar.events.search → writeResultTo `/data/calendar`，取当前会议 items/0。
- 展示用完整表达式 + JSON Pointer：`{{ ${/data/calendar/items/0/title} }}`、时间段一对 `{{ ... }}` 内拼接 dtStart/dtEnd。
- 初始 DataModel 仅根结构 + loading 态，不写死隐私数据。
- 事件：clickToDeeplink（settings / intelligent_scene_entry），仅在 DSL onClick，未进入 cardspec。
- refreshPlan onVisible/periodic(600s) 刷新当前会议状态。

## 校验
- genui 三行 JSONL 全部 json.loads 通过；cardspec JSON 通过。
- version v0.9、catalogId extended、surface/root/cardspec 尺寸均 140/2x2、root borderRadius 18 + clip 一致。
- 组件全部在 Form 10 组件内；children 仅组件 id；onClick.call 来自已声明 capability。
