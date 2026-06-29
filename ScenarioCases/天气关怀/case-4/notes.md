# case-4 天气关怀 · 深圳老家高温提醒 + 一键打电话

## 需求拆解
- 主答案：深圳老家当前温度（hero 主数值）。
- 支撑事实：天气状况 + 湿度（一行），极端天气提醒文案（独立一行，可标红）。
- 主动作：一键打电话给爸妈。

## 尺寸与布局
- 选 **2x4**（300x140）。理由：天气信息组 + 独立动作面板属于两个并列功能组，且 CTA 需要稳定热区，2x2 放不下并列关系。
- root：Row，padding 12 → 内容区 276x116，itemMargin 10。
  - 左 info_col 166 + 右 action_panel 88 + itemMargin 10 = 264 ≤ 276 ✓
- info_col（Column，166x116，itemMargin 6）：
  - head_row 40 + temp_value 42 + alert_line 18 + itemMargin 6×2=12 = 112 ≤ 116 ✓
  - head_row：temp_icon 36 + head_text_col 122 + itemMargin 8 = 166 ✓
- action_panel（Column，88x116，padding 10 → 内 68x96，itemMargin 8）：
  - call_icon 32 + call_hint 16 + call_btn 32 + itemMargin 8×2=16 = 96 ≤ 96 ✓
- CTA call_btn 68x32 ≥ 24vp 热区，14fp Medium 一行「打电话」。

## 数据契约
- 动态天气：capability `weather.overview.get`，`prefectureName=深圳市`、`forecastDays=1`，`writeResultTo=/data/weather`，bindingId `weather_now`。
- UI 路径：`/data/weather/items/weatherData/{temperature,weather_icon,humidity}`，均可由 outputSchema 推导。
- refreshPlan：onInstall/onVisible + 周期 1800s 刷新，盯紧温度变化。

## 极端天气标红实现
- `alertText` 与 `alertColor` 由端侧根据温度阈值（如 ≥35℃）归一化写入 DataModel。
- DSL 用原生 `{"path"}` 绑定 alert_line 的 content 和 color（兜底原因：颜色不能用整段 styles 表达式，且需端侧按阈值切换语义色）。
- 正常：`#99000000`（font_secondary）；极端高温：`#ffe84026`（warning，红，服务真实状态，符合状态色规则）。

## 一键打电话
- DSL `onClick` → `clickToCallPhone`，args `phonenumber` 来自 `/data/contact/parentPhone`（表达式读取）。
- 事件能力来自 click-event manifest，未写入 CardSpec。

## 颜色来源
- root 背景 `#ffffffff` = background_primary（light）。
- 主文字 `#e5000000` = font_primary；次要 `#99000000` = font_secondary。
- action_panel 背板 `#fff1f3f5` = comp_background_gray（中性背板承载动作组）。
- CTA `#ff0a59f7` = brand / background_emphasize；前景 `#ffffffff` = font_on_primary。
- 红色提醒 `#ffe84026` = warning，仅服务真实极端天气状态。
- 单一中性主色族 + 一个 brand 动作 + 一个状态红，未超色族限制。

## 信息职责互斥校验
- 城市（对象）/ 温度（主数值）/ 天气状况+湿度（环境支撑）/ 提醒（状态判断）/ 打电话（动作），各承载唯一职责，无单位换算或同义重复。

## 校验
- 模型内置校验通过：L0 协议、绑定路径可推导、L1 预算成立、颜色有来源、事件合法、尺寸一致。
- 无本地脚本运行环境（node/python 均缺失），故以人工预算核对为准。
