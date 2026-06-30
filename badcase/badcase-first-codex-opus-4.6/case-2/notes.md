# Case 2: 天气关怀卡片

## 需求摘要

深蓝到天空蓝渐变风格，显示深圳当前温度 31 度和天气情况，高温预警明显提醒，健康提示（感冒指数"易发"），一键拨打电话按钮联系家人。

## 尺寸选择

选择 `2x4`（300x140vp）。需要并排展示天气数据和健康指数，加上底部提示+通话按钮，`2x2` 无法容纳所有受保护文本。

## 布局预算

- root padding: 12vp，内容区 276x116vp
- topRow height: 72vp
  - weatherCol width: 120vp (cityWeather 12fp + tempRow 32fp + warningText 12fp)
  - divider: strokeWidth 1vp
  - healthCol width: 120vp
  - itemMargin: 10vp x 2 = 20vp
  - 横向: 120 + 1 + 120 + 20 = 261vp < 276vp，成立
- itemMargin between topRow/bottomRow: 8vp
- bottomRow height: 32vp
- 纵向: 72 + 8 + 32 = 112vp < 116vp，成立

## 横向预算（bottomRow）

- tipText width: 180vp
- callBtn width: 80vp
- itemMargin: 8vp
- 合计: 180 + 80 + 8 = 268vp < 276vp，成立

## 颜色来源

- root linearGradient:
  - start: `multi_color_02` light -> `#ff46b1e3`（天空蓝）
  - end: `multi_color_aux_02` light -> `#ff86c5e3`（浅天蓝）
  - 场景: 天气关怀/晴天高温，锚点：深圳天空、蓝色天幕、高温日光
- 文字 fontColor: `font_on_secondary` -> `#99ffffff`（次要信息）
- 温度/健康值: `font_on_primary` -> `#ffffffff`
- warningText: `warning` light -> `#ffe84026`（真实高温预警状态）
- callBtn bg: 半透明白 `#33ffffff`（on-color 系磨砂按钮）
- callBtn fontColor: `font_on_primary` -> `#ffffffff`
- divider color: `#66ffffff`（on-color 弱分隔）

## 场景拓展色板

未使用额外场景拓展色。root 渐变两个 stop 均直接来自官方多彩色 `multi_color_02` 和 `multi_color_aux_02`，属于同一蓝色族。

## 数据能力

使用 `weather.overview.get` 查询深圳天气。端侧结果写入 `/data/weather`，UI 读取 `temperature`、`weather_icon`、`high_temp` 等字段。

## 事件能力

底部按钮使用 `clickToCallPhone`，电话号码从 DataModel `/state/familyPhone` 读取（由端侧或用户配置写入）。

## 设计决策

- "毛玻璃质感" 通过渐变 root + 半透明按钮 + on-color 文字层级体现；Form 子集不支持 blur/backdrop-filter，用渐变和透明层模拟
- warningText 使用 `warning` 色标记真实高温预警，符合状态色使用规则
- 感冒指数"易发"不使用 warning/alert 色，因为它是健康建议而非紧急告警
- familyPhone 初始为空字符串，由端侧用户配置提供
