# 雨天打车 case-4 设计说明

## 用户意图拆解
- 主答案：深圳当前天气 + 温度（雨天，决定是否早出发）。
- 支撑事实：今天上午会议时间 + 会议名称（出发时间锚点）。
- 主动作：早点出发去打车。无打车 data/event capability，按规则不伪造 deeplink。

## 尺寸与布局
- 选 2x4：天气 hero 与会议信息为两个并列信息组，需同屏理解，2x2 放不下。
- split 结构：左天气列 118vp + 竖分隔 + 右列 138vp，itemMargin 12，root padding 12（300/140，内容区 276x116）。
- 左列：天气图标+文案行(28) / 温度 40fp 主数值 / 城市·降雨概率 12fp。
- 右列：会议背板(68，含时间行+标题) + 打车/查天气 CTA(36)，spaceBetween；68+8+36=112 ≤ 116，贴底约 12vp。
- 会议背板内：padding10，时间行18 + gap4 + 标题19 = 41 ≤ 48 可用高度。

## 动作取舍（重要）
- click-event manifest 仅支持 设置/天气/闹钟 deeplink 与 ViewCalendarEvent，无打车 app 目标。
- 不伪造打车 deeplink。CTA 改为「查看天气早出发」跳转华为天气 app（com.huawei.hmsapp.totemweather），与“看天气/决定早出发”意图一致且合法。
- 打车意图在文案语境中体现（早出发），不强行用错误路由满足。

## 颜色来源
- root 渐变：雨天蓝灰场景拓展色，scene=weather.rain.cool，baseSources=background_secondary/tertiary，低饱和蓝灰 #FF3F4A5C→#FF59647A，仅作 rootStart/rootEnd。
- 文本：饱和背景上用 on-color 白 #FFFFFFFF / #CCFFFFFF 次级。
- 会议背板：comp_background_secondary 类半透明白 #1FFFFFFF。
- 分隔线：comp_divider 类 #33FFFFFF。
- CTA：brand #FF0A59F7（合法品牌/导航强动作实色）。

## 数据契约
- 动态卡：weather.overview.get（深圳，forecastDays1）写 /data/weather；calendar.events.search（上午时段毫秒区间）写 /data/calendar。
- UI 路径：温度 /data/weather/items/weatherData/temperature；天气 weather_icon；降雨概率 weatherDailyData/0/dayRainProb；会议 /data/calendar/items/0/{dtStart,title}。
- 表达式优先读取 DataModel；初始 value 仅初始化根结构与 loading 态，不写死隐私数据。
- timeInterval 为占位毫秒区间（当天上午），端侧按本地时区与实际查询替换。

## 校验
- L0：三行 JSONL、v0.9、extended.catalog、2x4 尺寸一致、root shell 完整。✓
- L1：横向修正后 116(weatherCol)+1(divider)+134(rightCol)=251，+itemMargin10*2=20，=271 ≤ 内容区 276。✓
- 右列纵向：meetingPanel68 + itemMargin8 + taxiBtn36 = 112 ≤ 116，贴底约 12vp。✓
- 修正记录：初版列宽 118/138/itemMargin12 横向溢出 5vp，收窄为 116/134/itemMargin10。
- 受保护文本（温度/会议时间/标题/CTA）均显式宽度、无 ellipsis。✓
- 颜色全部可回溯。✓
