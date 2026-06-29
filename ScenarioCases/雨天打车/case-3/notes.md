# case-3 出差天气+会议卡片 布局与决策记录

## 场景
出差深圳参加客户会议。需要展示：深圳温度+城市位置、今天会议时间+名称，并提供一个动作入口避免耽误工作。

## 尺寸：2x4
信息需要两个并列理解的主对象（左侧天气、右侧会议+动作），2x2 容纳不下两组并列信息+动作，升级 2x4。
- root 300x140，padding 12，content 276x116，borderRadius 22，clip true。

## 横向预算（276）
- 左 weather_col width 150
- itemMargin 8
- 右 meeting_panel width 118
- 150 + 8 + 118 = 276 ✓

## 左列纵向（116, 居中）
- city_row 18（icon16 + 城市·天气文字）
- 6
- temp_text 40fp 主数值（绝对主答案，温度）
- 6
- weather_label 12fp 体感/湿度支撑
约 18+6+~44+6+14 = 88，居中留白合理。

## 右面板纵向（116, padding10 → 内容96x96）
- meet_time 14fp Semibold（emphasize 蓝）：时间是受保护文本，none 不截断
- 6
- meet_title 12fp，最多 2 行
- 6
- meet_btn 30vp 高，宽 98 与面板内容同宽，贴底
预算成立，按钮 >24vp 热区。

## 颜色（单一冷蓝场景族）
- root 渐变：weather/business 冷雾蓝场景拓展色，派生自 background_primary/secondary，低饱和高明度：#FFEAF1FB → #FFD7E5F6。
- 会议面板 background_primary 白 #FFFFFFFF（中性背板）。
- 时间/CTA 用 brand #FF0A59F7（font_emphasize / 品牌动作），CTA 前景 font_on_primary 白。
- 正文 font_primary #E5000000 / font_secondary #99000000。
单一主色族（蓝），状态色未滥用。

## 事件
- 打车意图无对应 event capability（manifest 仅有 clickToCallPhone / clickToDeeplink[设置/天气/闹钟] / clickToIntent[ViewCalendarEvent]），不可伪造打车跳转。
- 按钮改为「查看会议」，使用合法 clickToIntent ViewCalendarEvent，entityId 取首个日程，服务「不耽误工作」诉求（查看会议详情/地点）。

## 数据能力
- weather.overview.get（深圳市，forecastDays 1）→ /data/weather
- calendar.events.search（今日 timeInterval）→ /data/calendar
- DataModel 仅初始化根结构与 loading，不写死隐私数据。

## 信息职责（互斥）
- 温度（主数值）/ 城市+天气（位置+状态）/ 体感+湿度（支撑）/ 会议时间 / 会议名称 / 动作入口，各承载唯一事实，无重复。
