# case-1 马拉松赛事倒计时卡片 — 设计说明

## 取舍
- 主答案：距离开跑剩余天数（hero 大数字 40fp）。
- 支撑事实（≤2）：赛事名称（标题）、开跑日期+城市（meta 合并一行，避免拆成两个同义区块重复）。
- 主动作（1）：查看日程 → clickToIntent / ViewCalendarEvent（manifest 已声明）。
- 信息职责互斥：name=对象，daysLeft=时间差主指标，startDate+city=时间点+位置（合并），无重复单位/别名。

## 尺寸与布局（2x2 big-number-support）
- root 140x140，padding 12，内容区 116x116，borderRadius 18，clip。
- Column itemMargin 6，spaceBetween 纵向分配：
  - title 14fp ≈ 16vp 行高
  - hero 行固定 48vp（40fp 数字 + 12fp 单位，bottom 基线对齐）
  - meta 12fp ≈ 14vp
  - cta 30vp（≥24vp 热区），底边距 = 116 -(16+6+48+6+14+6+30)=... 由 spaceBetween 吸收，按钮贴底约 12vp 内。
- 受保护文本（标题/数字/日期/CTA）均 maxLines:1 + textOverflow:none，width 显式 116，不依赖默认伸缩。
- hero Row 数字+单位并排：40fp 数字“28”宽≈48vp，单位文字“天后开跑”5字≈60vp，+itemMargin4 = 112 ≤116，成立。

## 颜色来源
- root 渐变：brand(#FF0A59F7 light) → multi_color_08 同族(#FF317AF7)，temporal-band 线性渐变，服务“运动/赛事/倒计时”场景，单一主色族（蓝）。
- 前景：on-color。主文本 font_on_primary #FFFFFFFF；次要文本 font_on_secondary #99FFFFFF。
- CTA 背板 comp_background_secondary on-color 近似 #33FFFFFF（饱和背景上中性磨砂层），文字 on-primary。

## 校验
- L0：三行 JSONL，version v0.9，catalog extended，2x2 尺寸一致。
- 绑定：所有 {{ }} 表达式引用 /race/* 均在 updateDataModel.value 中存在。
- 事件：clickToIntent + ViewCalendarEvent 来自已声明 manifest，entityId 端侧提供（初始空串）。
- 静态卡片：cardspec 只含 suggestSize，无虚构 dataBindings。
