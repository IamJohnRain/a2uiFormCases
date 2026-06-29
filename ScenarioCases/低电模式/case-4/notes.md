# case-4 低电模式提醒卡片 — 设计说明

## 需求拆解
- 主答案：当前电量百分比（实时提醒还剩多少电）。
- 触发场景：电量低于 20% 时提醒（属于真实告警状态，可使用 warning 状态色）。
- 主动作：一键进入省电模式。

## 信息职责（互斥，无重复）
1. 状态行（图标 + “电量偏低”）：状态判断，唯一承载“低电告警”这一事实。
2. Hero 数值（`18%`）：主指标，唯一承载“还剩多少电”。
3. CTA 按钮（进入省电模式）：唯一动作入口。

未复述：进度条/第二个百分比标签未加入，避免与 hero 数值构成事实等价类重复。

## 尺寸与布局（2x2，内容区 116x116）
- root：Column，padding 12，justifyContent spaceBetween，itemMargin 8。
- 纵向预算：状态行 22 + 间距 8 + hero 数值 48 + 间距 8 + CTA 32 = 118 ≈ 116（spaceBetween 自适应分布，底部贴近安全区底边）。
- 状态行：width 116，图标 18 + gap 6 + 文本（4 字 ×14 ≈ 56），总和 < 116，文本不截断。
- hero 数值：40fp，`18%` 约 2×24 + 24 ≈ 72vp < 116，完整显示。
- CTA：width 116（与内容同宽，共享左右边界），height 32 ≥ 24vp 热区，圆角 16（height/2 胶囊）。

## 颜色来源
- root 背景：`background_secondary` light `#fff1f3f5`（中性表面）。
- 状态文本：`warning` light `#ffe84026`，服务真实低电告警状态。
- hero 数值：`font_primary` light `#e5000000`（主信息，保持中性高对比，避免双高饱和前景）。
- CTA 背景：`brand` light `#ff0a59f7`，进入省电模式属于明确导航动作，可用品牌实色。
- CTA 文本：`font_on_primary` `#ffffffff`，品牌背景上使用 on-color。
- 单一主场景色族（中性 + brand 动作 + warning 状态语义），无第二高饱和主色。

## 事件能力
- CTA 使用 manifest 中 `clickToDeeplink`，目标为设置应用 `intelligent_scene_entry`（情景模式/省电相关场景页），为 supportedTargets 中合法组合。无独立“省电模式”target，故选最贴近的合法情景模式页。

## 数据契约
- 电量为系统实时值，端侧通过 `updateDataModel` 推送 `/data/battery/level`。
- data-capability 目录未声明电量能力，故 CardSpec 不虚构 `dataBindings`/`refreshPlan`，仅保留 `suggestSize: 2x2`。
- DataModel 初始化 `/data/battery/level` 与 `/state/loading` 根结构，level 用占位值 18（< 20，符合提醒场景），非真实隐私数据。

## 校验
- L0 协议：三行 JSONL、v0.9、extended catalog、2x2 尺寸一致、root shell 完整。✓
- L1 布局：纵向/横向预算成立，受保护文本（状态、数值、CTA）完整。✓
- L2 内容：一个主答案 + 一条状态 + 一个动作，信息职责互斥。✓
- 颜色：全部可回溯官方 token。✓
