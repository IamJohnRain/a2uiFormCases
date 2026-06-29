# case-5 雨天打车卡片 — 布局与校验记录

## 需求拆解
用户意图：下雨天步行怕迟到，想看 (1) 深圳当前天气+温度，(2) 今日日程与会议时间，(3) 一键叫车去公司。
三组并列信息（天气 hero / 日程列表 / 动作）无法压进 2x2，升级 **2x4**。

## 尺寸与分区（2x4，内容区 276×116）
- root：Row 300×140，padding 12，borderRadius 22，clip。
- 左列 leftCol 144：天气区（城市+小雨 / 温度）+ 底部叫车 CTA。
- 竖分隔 midDivider 1。
- 右列 rightCol 108：今日日程标题 + 日程列表（模板循环）。
- 横向预算：144 + 1 + 108 + itemMargin 10×2 = 273 ≤ 276 ✓

## 关键子项预算
- wxRow 144：icon 22 + gap 6 + wxText 116 = 144 ✓
- tempRow：40fp 主温度 + 16fp 单位（底对齐），唯一绝对主数值用 40fp。
- taxiBtn 144×34：car 图标 18 + gap 6 + 文本，居中；CTA 14fp Medium，热区高度 34≥24 ✓
- 右列 schTitle：icon 18 + gap 6 + 标题 84 = 108 ✓
- schList 88 高度容纳 2 条日程（每条 time 12fp + title 12fp ≈ 30 + itemMargin 8）。

## 数据契约
- 动态：weather.overview.get（prefectureName=深圳市，forecastDays=1）→ /data/weather；calendar.events.search（今日 timeInterval）→ /data/calendar。
- 温度用表达式读取 weatherData.temperature；城市名用表达式拼接。
- 日程为数组，用模板循环 /data/calendar/items；模板内相对路径 dtStart/dtEnd/title/entityId。
- updateDataModel 仅初始化根结构 + loading 态，不写死隐私数据。

## 事件
- 日程项 onClick = clickToIntent / ViewCalendarEvent，entityId 取当前项相对路径（已声明能力）。
- **叫车动作**：click-event manifest 中无打车/网约车 supportedTarget，不能伪造 deeplink。
  因此 CTA 改为静态视觉胶囊（Row + 图标 + 文案），不挂伪造 onClick，避免误导跳错应用。
  若端侧补充打车 event-capability，再为 taxiBtn 接入对应 onClick。

## 颜色来源
- root 背景 #FFEEF1F5 ← background_secondary 雨天蓝灰浅表面族（场景 weather.rain，低饱和高明度）。
- 主文字 #E5000000 = font_primary；次文字 #99000000 = font_secondary。
- 时间高亮 #FF0A59F7 = font_emphasize/brand；CTA 实色 #FF0A59F7 = comp_background_emphasize（导航类强动作允许实色）。
- 分隔线 #22000000 低 alpha 黑；CTA 前景 #FFFFFFFF = font_on_primary。
- 单一主场景色族（蓝），无第二高饱和色。

## 模型内置校验
- L0 协议：三行 JSONL，version/catalog/尺寸一致，仅用 Form 组件，children 合法，模板对象仅 componentId+path ✓
- 绑定：所有可见路径均可由 writeResultTo+outputSchema 推导 ✓
- L1 布局：横纵预算均不超父容器，受保护文本（城市/温度/时间/CTA）完整显示 ✓
- 颜色：均可回溯 token ✓
- 事件：仅用已声明能力，未伪造打车跳转 ✓
