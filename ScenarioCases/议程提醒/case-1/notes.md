# case-1 最近日程一键入会卡片 — 设计 / 校验记录

## 场景与取舍
- 主答案：最近一个即将开始的会议（`/data/calendar/items/0`）。
- 支撑事实（≤2）：会议时间段（dtStart-dtEnd）、地点（eventLocation）。
- 主动作（1）：一键入会 → `clickToIntent / ViewCalendarEvent`，参数 entityId 取当前日程。
- 信息职责互斥：状态pill=即将开始；title=对象；timeRow=时间点/段；location=位置；CTA=动作入口。无重复事实。

## 尺寸与布局（2x2，paper-panel / top-hero-pill 变体）
- surface/root/CardSpec 尺寸一致：140x140，root borderRadius 18、clip true、padding 12。
- 内容区 116x116。root = Column，spaceBetween：上部信息组 + 底部 CTA 贴底。
- topGroup（width 116，itemMargin 4）：
  - status 10fp（弱提示）≈12vp
  - title 14fp 600 ≈16vp
  - timeRow 12fp ≈14vp
  - location 12fp ≈14vp
  - 组内 3 个间隔 *4 = 12vp；总高 ≈ 12+16+14+14+12 = 68vp
- CTA：width 116、height 30、borderRadius 15（pill=height/2）。
- 纵向预算：68(top) + 30(cta) = 98vp ≤ 116vp，spaceBetween 余 18vp 作组间呼吸；CTA 底边距 = padding 12vp（8-12 区间内，贴底）。
- 横向：所有元素 width 116 = 内容区宽，统一左右边界。

## 颜色来源
- root 背景 `#FFFFFFFF` = comp_background_primary(light) / background_primary。
- title/timeRow 文字 `#E5000000` = font_primary(light)。
- location `#99000000` = font_secondary(light)。
- status `#FF0A59F7` = font_emphasize / brand(light)，仅作即将开始的轻强调。
- CTA 背景 `#FF0A59F7` = brand(light)，前景 `#FFFFFFFF` = font_on_primary（品牌背景配 on-color）。
- 单一主场景色族（brand 蓝），无第二高饱和前景。

## 数据契约
- 动态：CardSpec dataBindings = calendar.events.search，timeInterval 为今日起止毫秒占位（端侧按本地时区替换）。
- writeResultTo `/data/calendar`；UI 路径 `/data/calendar/items/0/*` 由 outputSchema 推导。
- 初始 DataModel 仅含 `/data/calendar/items: []` 与 `/state/loading: true`，不写死真实日程。
- refreshPlan 引用已声明 bindingId `upcoming_meeting`（onInstall/onVisible/periodic 1800s）。

## 校验
- L0 协议：三行 JSONL（createSurface/updateComponents/updateDataModel），catalogId extended，组件均在 Form 子集（Column/Text/Button）。
- 绑定：所有表达式路径与事件 entityId 均可由 writeResultTo+outputSchema 推导。
- 事件：onClick.call=clickToIntent，intentName=ViewCalendarEvent，参数符合 manifest。
- 受保护文本：time/CTA 不截断；title 长度不可控，单行 ellipsis 作兜底（地点同）。
