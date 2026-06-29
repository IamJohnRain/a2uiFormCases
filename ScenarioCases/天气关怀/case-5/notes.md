# 天气关怀卡片 — case-5 设计与校验记录

## 取舍
- 主答案：深圳当前温度 31°（hero）。
- 支撑事实：天气状况（晴）、高温预警提醒、感冒指数（易发）。
- 主动作：一键拨打电话（clickToCallPhone）。
- 信息职责互斥：温度=主数值；城市·天气=对象/状态行；高温预警=状态提醒胶囊；感冒指数=健康判断面板；按钮=动作入口。无重复事实。

## 尺寸
- 选 2x4（300x140，root borderRadius 22, clip）。理由：温度 hero + 天气 + 高温预警 + 健康提示 + 拨号按钮，存在两个并列信息组与一个动作区，超出 2x2 可读容量。
- 内容区 276x116。左列 158 + itemMargin 10 + 右列 108 = 276 ✓。

## 布局预算
- 左列（158x116）：城市行 18 + 8 + 温度行 44 + 8 + 预警胶囊 30 = 108 ≤ 116 ✓。
- 温度行：40fp 主数值 + 32fp 度符号，bottom 对齐。
- 预警胶囊：高 30，padding 上下 6 → 内容 18；圆点 8 + gap 6 + 文本 118 + padding 左右 10*2 = 152 ≤ 158 ✓。
- 右列（108x116）：健康面板 68 + itemMargin 10 + 按钮 38 = 116 ✓，按钮贴底（底边距≈0，因右列高=内容区高，整体 root padding 12 已是安全区底）。
- 健康面板 padding 10：标题 16 + gap 6 + 值 22 = 44，加 padding 20 = 64 ≤ 68 ✓。
- 按钮高 38 ≥ 24vp 热区；与上方面板等宽 108，共享左右边界。

## 颜色来源
- root 渐变（ambient-band，天气场景，线性 145°）：
  - `#FF0A4DC2` 深蓝：multi_color_08(`#0a59f7`) 同族压暗派生 rootStart。
  - `#FF0A59F7` = multi_color_08 / brand。
  - `#FF46B1E3` = multi_color_02（天空蓝）rootEnd。
  - scene=weather.daytime.warm，baseSources=multi_color_08/02，单一主色族（蓝）✓。
- 毛玻璃面板：白色低 alpha `#26FFFFFF`（中性磨砂层，类比 comp_background_secondary 白模），边框 `#33FFFFFF`/`#4DFFFFFF`。
- 高温预警点：`#FFED6F21` = alert（真实高温状态），放在中性玻璃胶囊内，未作主题主色 ✓。
- 文本：饱和渐变背景上用 on-color（白/半透明白）：`#FFFFFFFF`、`#CCFFFFFF`。
- 按钮：白底 + brand 蓝字 `#FF0A59F7`，强动作实色。

## 校验
- L0：三行 JSONL（createSurface/updateComponents/updateDataModel）；version v0.9；catalog extended；2x4 尺寸一致；root 为已存在组件且含 width/height/padding/borderRadius/clip/渐变。
- 绑定：所有表达式路径（weather.*, health.*, action.*）在 updateDataModel.value 中存在 ✓。
- 事件：clickToCallPhone 来自 manifest，参数 phonenumber 合法 ✓。
- 组件：仅用 Row/Column/Stack/Text/Button，均在 Form 子集内。

## 改进点（备选）
- 若需真实拨打父母号码，将 action.phone 替换为实际号码。
- warnDot 用 Stack 占位点表示状态色，避免文本承载色块。
