# case-3 低电模式卡片 notes

## 需求
显示当前电量；电量低时提示开启省电模式；提供快速进入省电模式的按钮。

## 信息取舍（2x2）
- 主答案：当前电量数值（40fp 主数值 + 18fp 百分号）。
- 支撑事实：低电提示文案（“电量偏低，建议开启省电模式”）。
- 主动作：开启省电模式按钮。
- 每个事实只由一个组件承载，无重复。

## 布局预算（root padding 12，内容区 116x116）
- root Column，itemMargin 8，spaceBetween 纵向分布。
- header 文本 12fp，约高 14。
- levelRow 高 42，承载 40fp 数字（底对齐 bottom）+ 百分号。
- tipText 宽 116，12fp，maxLines 2，足够显示一行半中文。
- saveBtn 宽 116、高 34（≥24vp 热区），圆角 17，贴底。
- 纵向估算：14 + 42 + (12*2) + 34 = 约 114，加 itemMargin 后由 spaceBetween 弹性分布，未超 116。

## 事件能力
- clickToDeeplink → 设置 intelligent_scene_entry（情景模式/省电入口），来源 click-event.md manifest 合法目标。

## 颜色来源
- 背景 #FFFFFFFF = background_primary(light)。
- 主数值/百分号/按钮填充 #FFED6F21 = alert(light)，服务真实低电状态。
- 标题/提示 #99000000 = font_secondary(light)。
- 按钮文字 #FFFFFFFF = font_on_primary（饱和背景上用 on-color）。
- 单一状态色族，未叠加第二高饱和前景。

## 校验
- L0：三行 JSONL，version v0.9，catalog extended，2x2 尺寸一致，root 含 width/height/padding/borderRadius/clip/backgroundColor。
- L1：并排/纵向预算成立，受保护文本（电量、CTA）完整。
- 数据：battery.level 在 updateDataModel 初始化为 18。
