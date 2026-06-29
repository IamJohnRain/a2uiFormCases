# 睡眠卡片 case-3 设计说明

## 需求拆解
- 主答案：睡眠不足提醒（状态行 + 实际时长支撑事实）。
- 主动作：设闹钟按钮 → 跳转闹钟应用。
- 派生事实：根据睡眠数据自动推荐的最佳入睡时间（hero 主数值）。

## 尺寸与布局
- `2x2`（140×140，内容区 116×116），三段纵向：
  - 状态行 20vp：`睡眠不足`（左，white 600）+ 时长 `5h42m`（右，white60）。spaceBetween。
  - 推荐入睡块 48vp：弱标签 10fp + 入睡时间 32fp 主数值（受保护，完整显示）。
  - 设闹钟按钮 32vp：宽 116vp（与上方内容同宽，贴底），圆角 16 pill。
- itemMargin 8 + padding 12：20+48+32 = 100，加 2 个 8vp 组间距 = 116，等于内容高，预算成立。

## 颜色（scene: sleep.night）
- root 线性渐变：`#FF2A2540`(夜灰紫) → `#FF564AF7`(multi_color_01)，angle 160。
  - baseSources：`multi_color_01` + `background_tertiary(dark)` 派生暗夜表面，符合睡眠夜间推荐方向。
- 前景全部 on-color：标题/主数值 `#FFFFFFFF`，弱信息 `#99FFFFFF`。
- 按钮中性磨砂背板 `#33FFFFFF`（comp 透明层），不引入第二高饱和前景。
- 状态色未误用 warning/alert：睡眠不足是常规提醒，用 on-color 文本承载。

## 数据契约
- 无端侧睡眠 data-capability 声明，"自动推荐入睡时间" 与 "实际时长" 由端侧/agent 计算后写入 DataModel；
  CardSpec 仅输出 `suggestSize`，不虚构 dataBindings / refreshPlan。
- 展示值用完整表达式读取 `/sleep/durationLabel`、`/sleep/recommendedBedtime`。

## 事件
- `clickToDeeplink` → 闹钟 app（bundleName com.huawei.hmos.clock / abilityName .phone / uri 空），来自 click-event manifest 合法目标。

## 校验
- L0：三行 JSONL，version v0.9，catalogId extended，2x2 尺寸/圆角一致，root shell 完整。
- L1：纵向预算闭合，受保护文本（时长/入睡时间/CTA）完整，按钮 ≥24vp 且贴底（底边距 12vp）。
- L2：信息职责互斥（状态/时长/推荐时间/动作各一），无同义重复。
