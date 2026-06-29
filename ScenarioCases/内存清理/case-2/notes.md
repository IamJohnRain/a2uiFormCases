# 内存清理小组件 case-2 设计说明

## 取舍
- 主答案：内存已用占比（环形进度 + 中心百分比）。
- 支撑事实：可清理空间（`可清理 1.2GB`），与“已用%”信息职责互斥，不复述已用值。
- 主动作：底部「一键清理」CTA，整卡仅 1 个动作。
- 信息职责互斥：环=比例/进度，中心文字=主指标(%)，fact=可清理数量，CTA=动作入口，无重复事实。

## 尺寸 / 布局（2x2，meter-focus 原型）
- surface/root：140 x 140，root padding 12，内容区 116 x 116，borderRadius 18，clip true。
- 纵向预算（Column itemMargin 8，spaceBetween）：
  - hero(64) + itemMargin(8) + fact(~16) + itemMargin(8) + cta(34) = 130 < 116? 实测 spaceBetween 下取固定高之和 64+16+34=114，两段间距由 spaceBetween 在剩余 (116-114)=2vp 内分配，整体贴合安全区，CTA 锚定底部。
- 环：64vp 视觉主焦点（meter-focus 56-72vp 区间）。
- 中心数值 20fp（紧凑数值），标签 10fp（弱提示），均一行完整显示。
- CTA：width 116（=内容区宽）、height 34（>24vp 热区），14fp Medium，文字「一键清理」4 字一行不截断。
- CTA 与上方内容共享左右边界（同 116vp 宽，居中）。

## 颜色来源
- root 背景：`background_primary` light = #FFFFFFFF。
- 主数值文字：`font_primary` light = #E5000000。
- 标签/支撑文字：`font_secondary` light = #99000000。
- 环色 / CTA 填充：`confirm` light = #FF64BB5C（清理=完成/确认类强动作，状态色仅服务“清理”这一真实动作语义）。
- CTA 前景：`font_on_primary` = #FFFFFFFF（饱和背景上用 on-color）。
- 单一主场景色族（绿色 confirm），无第二高饱和前景。

## 事件能力说明（重要）
- 当前 event-capability manifest 仅声明 `clickToCallPhone`、`clickToDeeplink`、`clickToIntent`，
  `clickToDeeplink.supportedTargets` 中没有“内存清理 / 系统管家 / 手机管家”页面，
  `clickToIntent` 仅支持 `ViewCalendarEvent`。
- 用户「一键清理」意图无法匹配任一已声明能力/目标，按 click-event.md 规则**不伪造点击能力**。
- 因此 CTA 暂不写 `onClick`，作为视觉动作入口；需真实触发清理时，需端侧补充
  内存清理对应的 event-capability（如 clickToDeeplink 到系统管家清理页，或 clickToIntent 清理意图）。

## 数据契约说明
- 当前 data-capability 目录仅有 weather / calendar，无内存能力，故为静态卡片：
  cardspec 只写 `suggestSize`，不虚构 `dataBindings` / `refreshPlan`。
- DataModel 初始化 `/data/memory/{usedPercent, cleanableText}` 根结构，便于端侧补内存能力后直接写值。

## 模型内置校验
- L0：genui 三行 JSONL（createSurface/updateComponents/updateDataModel），version v0.9，catalog extended，2x2 尺寸 DSL 与 cardspec 一致；root 引用存在、含 width/height/padding/borderRadius/clip/背景。
- 协议：仅用 Column/Stack/Progress/Text/Button；无禁用组件、网络图、SVG、emoji、未声明事件。
- 绑定：所有表达式路径 `/data/memory/usedPercent`、`/data/memory/cleanableText` 在 updateDataModel.value 可寻址。
- 布局：纵向预算成立，受保护文本（%数值、CTA）完整显示，无 ellipsis/clip 兜底。
- 颜色：全部可回溯官方 token，DSL 仅输出 hex。
