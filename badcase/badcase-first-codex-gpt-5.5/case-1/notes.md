# 会议日程卡片记录

- 布局选择：使用 2x4，原因是会议名称是受保护动态文本，2x2 同时容纳图标标题、会议名称/时间、提醒和 CTA 时容易挤压；横版三段结构保证名称可用两行显示，按钮贴底。标题行使用素材库会议图标 `resource/meeting_widget/icon_meeting.png`。
- DataModel：动态可见值均优先使用完整表达式读取 `/data/calendar/items/0` 与 `/reminder/minutes`；CardSpec 使用 `calendar.events.search` 查询 2026-06-30 下午会议并写入 `/data/calendar`。
- 行为链：底部按钮只在 DSL 中声明 `clickToDeeplink`，目标为设置应用 `intelligent_scene_entry`，用于打开情景/专注模式入口；CardSpec 不写点击行为。
- 自检：组件限定在 Form 允许集内；surface/root/CardSpec 均为 2x4；root shell 完整；图片为素材库资源；颜色来自 HarmonyOS brand、font 与 comp emphasis token；底部按钮 116x28vp，满足热区要求。
