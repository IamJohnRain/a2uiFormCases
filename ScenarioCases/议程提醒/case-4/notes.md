# case-4 一键入会卡片 — 布局与校验记录

## 需求拆解
- 主答案：下一个会议（名称 + 时间 + 会议号）。
- 主动作：一键入会按钮，点击进入会议/日程详情。
- 辅助：顶部会议 logo + "会议日程" 标识；右上角圆形角标显示待处理会议数量。

## 尺寸与布局（2x2，140x140，安全区 116x116）
采用 `top-hero-pill` 原型 + 角标叠加（Stack）：
- root: Stack（alignContent=topEnd），承载 main 内容列 + badge 角标层。
- main: Column spaceBetween，三段：
  - header(20vp)：logo(18) + "会议日程"(12fp/secondary)
  - hero(58vp)：会议名(16fp/600/primary) + 时间(12fp/secondary) + 会议号(12fp/tertiary)，itemMargin 3
  - cta(30vp)：一键入会按钮，full-width 116vp，圆角 15
- 纵向预算：20+58+30=108，剩余 8vp 由 spaceBetween 分到 2 个组间隙（各 4vp）；CTA 距卡片底边 12vp（root padding），落在 8-16vp 区间。
- 角标 badge：20x20 圆形（borderRadius 10），叠在右上角，不覆盖受保护文本（位于 hero 上方右侧的 header 行外缘）。

## 文本宽度自检（116vp 可用宽）
- 会议名 16fp 单行：中文约 16vp/字，约 7 字内安全，溢出由真实数据决定，受保护单行不省略。
- 时间 "HH:MM - HH:MM" 约 11 字符×7≈77vp < 116。
- 会议号行 12fp tertiary < 116。

## 颜色来源
- root 背景：background_primary → #FFFFFFFF（浅色）。
- 文字：font_primary #E5000000 / font_secondary #99000000 / font_tertiary #66000000。
- CTA 填充：brand #FF0A59F7（品牌强动作实色），前景 font_on_primary #FFFFFFFF。
- 角标：alert #FFED6F21（multi_color_09 同色），作真实"待处理会议数量"通知计数，仅用于 20vp 小角标，前景 #FFFFFFFF。单一主场景色族=brand，角标为状态计数，受控使用。

## 数据契约
- 动态：CardSpec dataBindings 调用 calendar.events.search（writeResultTo=/data/calendar）。
- UI 路径：/data/calendar/items[0] 的 title / dtStart / dtEnd / description / entityId。
- 会议号映射到 description（outputSchema 无专用 ID 字段，取自由文本字段兜底）。
- 角标数量：size($__dataModel.data.calendar.items)。
- 入会动作：clickToIntent + ViewCalendarEvent，entityId 取 items[0]。
- 初始 DataModel：items=[]，state.loading=true，不写死真实日程。

## 校验
- L0：三行 JSONL；version v0.9；catalog extended；2x2 尺寸 root/surface/cardspec 一致；root 含 width/height/padding/borderRadius/clip/backgroundColor。
- L1：纵向/横向预算成立；CTA 显式宽高；badge ≥24vp 不适用（非点击元素，仅展示）。
- L2：信息职责互斥（对象/标题/时间/编号/数量/动作各一）；单一品牌色族 + 状态计数；无 emoji/SVG/网络图。
