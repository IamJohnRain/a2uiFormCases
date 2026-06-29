# 会议日程免打扰卡片 — case-1 notes

## 场景与取舍
- 主答案：当前正处于会议时段，免打扰/专注模式已开启。
- 支撑事实（≤2）：会议标题、会议时间段（dtStart - dtEnd）。
- 主动作（1）：跳转系统设置「情景模式」入口，管理专注/免打扰模式。
- 每个组件单一职责：状态徽标=状态，标题=对象，时间=时间段，按钮=动作入口。无重复事实。

## 尺寸与布局（2x2）
- root 140x140，padding 12，内容区 116x116，borderRadius 18，clip true。
- 纵向 spaceBetween 三段：
  - statusRow 20vp（图标16 + 6gap + 文本12fp）。
  - hero 中部：标题16fp(最多2行) + 4gap + 时间14fp。
  - action 按钮 32vp，宽 116 与内容同宽，贴底（spaceBetween 自然锚定，底边距由内容高度决定，<16vp）。
- 横向预算：statusRow 16+6+文本 ≤116；按钮宽 116=内容区宽，热区 116x32 远大于 24vp。
- 受保护文本：状态、标题、时间、CTA 均 textOverflow:none，无截断。

## 颜色来源
- root 渐变：brand 深蓝族场景拓展（focus.calm），baseSources=brand(#0a59f7)/multi_color_08，派生低饱和深蓝 rootStart #FF1B2A4A → rootEnd #FF2A3F6E，仅用于 root 表面，营造专注/勿扰沉静氛围。
- 前景：饱和深色背景上使用 on-color 前景：font_on_primary #FFFFFFFF（标题/状态/CTA），font_on_secondary #99FFFFFF（时间次要文本）。
- 按钮背板：comp 中性透明白 #33FFFFFF（on-dark 弱面），非第二高饱和色。
- 单一主场景色族（brand 蓝），无状态色滥用。

## 数据与事件
- 动态：calendar.events.search 查询今日日程写入 /data/calendar，UI 读 /data/calendar/items/0 的 title/dtStart/dtEnd。
- 初始 DataModel 用占位（加载中…/--:--）+ state.loading，不写死真实隐私。
- 事件：DSL onClick clickToDeeplink → 设置 intelligent_scene_entry（manifest 合法目标，对应情景模式/免打扰/专注）。CardSpec 不含事件。
- 图标：resource/schedule_widget/icon_focus.png（专注语义，素材库声明）。

## 校验（模型内置）
- L0 协议：三行 JSONL、v0.9、extended catalog、2x2 尺寸一致、root shell 完整、仅用支持组件、表达式完整单对 {{}}。
- L1 布局：横纵预算成立，按钮显式宽高，受保护文本完整。
- 颜色：均可回溯 token / 场景拓展，DSL 仅输出 hex。
- 信息职责互斥，无事实等价类重复。
