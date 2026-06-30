# Case 1: 会议日程卡片

## 需求摘要

顶部会议图标 + "会议日程" 标题，中间显示下一个会议名称和时间，下方小字显示提前多少分钟提醒，底部按钮点击开启专注模式。

## 尺寸选择

选择 `2x2`（140x140vp）。内容为标题行 + 会议名/时间 + 提醒/按钮，三段结构完全符合 2x2 的 top-hero-bottom 原型。

## 布局预算

- root padding: 12vp，内容区 116x116vp
- headerRow: 20vp（icon 20vp + title 14fp）
- itemMargin between regions: 6vp x 2 = 12vp
- infoCol: meetingName 16fp(~18vp line) + 4vp gap + meetingTime 12fp(~14vp line) = ~36vp
- bottomRow: 28vp（Button height 28vp，remindText 10fp 居中）
- 总计: 20 + 6 + 36 + 6 + 28 = 96vp < 116vp，预算成立

## 横向预算（bottomRow）

- remindText width: 50vp
- focusBtn width: 60vp
- itemMargin: 6vp
- 合计: 50 + 60 + 6 = 116vp = 父容器宽度，精确匹配

## 颜色来源

- root bg: `background_primary` light -> `#ffffffff`
- title/meetingName fontColor: `font_primary` light -> `#e5000000`
- meetingTime fontColor: `font_secondary` light -> `#99000000`
- remindText fontColor: `font_tertiary` light -> `#66000000`
- focusBtn bg: `comp_emphasize_tertiary` light -> `#190a59f7`
- focusBtn fontColor: `brand` light -> `#ff0a59f7`

## 数据能力

使用 `calendar.events.search` 查询今日日程。端侧将结果写入 `/data/calendar`，UI 读取 `nextMeeting` 字段（端侧归一化后的下一条会议）。

## 事件能力

底部按钮使用 `clickToDeeplink` 跳转到系统设置的情景模式页面（`intelligent_scene_entry`），用户可在此开启专注/免打扰模式。

## 改进点

- remindLabel 由端侧根据日程 remindTime 字段计算后写入 state，避免前端硬编码
- nextMeeting 结构由端侧从 items 数组取最近一条归一化提供
