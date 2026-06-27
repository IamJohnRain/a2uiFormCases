# Case 7 Notes

## 布局理由

- 场景/模式：模式 1，新建综合功能卡片；语义覆盖 status glance、time/reminder、action card。
- 尺寸：选择 `2x4`，因为天气温度、即将到来的事项、时间提醒和一键打车同时出现，`2x2` 会挤压受保护的温度、时间和 CTA。
- 语义角色：identity 是“天气 / 即将到来”；primary answer 是当前温度；metric 是温度和事项时间；context 是天气状态、当日摘要、事项地点和提醒文案；action 是“打车”。
- 主区域：横向 3 个 pane，左侧天气温度，中间日程提醒，右侧打车动作，符合 `2x4` 主区域 <= 4 的限制。
- 视觉焦点：左侧大号温度是唯一主焦点；中间半透明事项块作为次级信息；右侧白色按钮保留动作可见性但不抢主信息。
- 信息节奏：从“现在天气”到“下一步事项”再到“出发动作”，形成通勤前的扫视顺序。
- 关键横向关系：天气和日程并列，打车依附于日程目的地；没有把 CTA 和多个标签塞进同一行。
- 必须完整显示：温度、天气状态、日程时间、日程标题、提醒标签、CTA“打车”和状态“出发”均不使用 ellipsis；只有地点是次要字段，允许 ellipsis。

## Row 宽度预算

- root：320 - padding 24 = 296vp；weatherPane 96 + schedulePane 126 + ridePane 50 + gaps 16 = 288vp，可容纳，且三列均有明确宽度。
- scheduleHeader：126vp，左右无额外 padding，gap 5；`即将到来` 约 58vp，`提醒` badge 约 34vp，合计 97vp，安全。
- eventMain：eventItem 内宽约 126 - 14 = 112vp，gap 5，time 42vp，title 弹性约 65vp；时间 `HH:MM` 完整，标题使用 `minFontSize` 并不截断关键文本。

## 显式改进

第一版内部草稿把打车按钮和提醒标签都放在中间详情行，导致日程时间、标题和 CTA 争抢同一行宽度。已改为三 pane：右侧独立动作列，中间只承载日程和提醒，降低拥挤度并保护 CTA 与时间文本。

## DataModel 与 CardSpec

- `weather.overview.get` 写入 `/data/weather`，参数 `forecastDays: 1`，UI 读取 outputSchema 内的 `current.temperatureText` 与 `current.condition`；天气提示文案放在 `/state/weatherHint`，不伪装成端侧能力输出。
- `calendar.events.search` 写入 `/data/calendar`，参数 `range: "next24Hours"`、`limit: 2`，UI 通过模板读取 `/data/calendar/items`。
- `suggestSize` 为 `2x4`，与 DSL root `320 x 160` 一致。
- `refreshPlan` 在安装、可见和 1800 秒周期刷新两个 binding。

## 宿主函数假设

- `requestRide` 是宿主 catalog 已声明的自定义函数。
- `requestRide` 接收 `pickupHint`、`destinationHint`、`calendarBinding`，可用当前定位和最近日程地点解析打车起终点。
- 打车不在本 skill 已声明 CardSpec data capability 中，因此不虚构 dataBinding，只通过 Button `onClick` 触发宿主函数。

## 校验结果

- `validate_genui_card.py`：通过，消息数 3，组件数 21，surfaceId `case7-commute-brief`。
- `validate_cardspec.py`：通过。
