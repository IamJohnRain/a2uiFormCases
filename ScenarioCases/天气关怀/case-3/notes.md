# case-3 父母关怀卡片 — 布局与校验记录

## 需求
- 显示深圳温度与感冒指数。
- 一键给爸爸（15012345678）打电话。

## 尺寸与布局
- 选 2x2（140x140，root padding 12，内容区 116x116，borderRadius 18，clip）。
- 一个主对象 + 一条支撑事实 + 一个动作，符合 2x2 容量上限。
- 纵向 Column（itemMargin 8），spaceBetween：
  - header 标题/位置：16vp
  - temp_row 温度 hero：40vp（40fp 主数值 + 18fp 单位，bottom 对齐）
  - cold_panel 感冒指数背板：24vp（中性背板 #19000000）
  - call_btn CTA：32vp，宽 116 与内容同宽，圆角 16（height/2）
  - 高度预算：16+40+24+32 = 112 + itemMargin 8*3=24 → 用 spaceBetween 吸收，底边贴安全区。
- CTA 为受保护文本「给爸爸打电话」6 字 14fp，一行可容纳于 116vp。

## 数据
- 温度来自 weather.overview.get（深圳市，forecastDays 1），路径 /data/weather/items/weatherData/temperature，表达式读取。
- 感冒指数非天气 outputSchema 字段，作为 DataModel 端侧维护值 /data/care/coldIndex 展示，初始「易发」。
- 电话号码 /data/care/fatherPhone 初始 15012345678。

## 事件
- clickToCallPhone（manifest 已声明），args.phonenumber 用表达式读取 DataModel，未写入 CardSpec。

## 颜色（来源）
- root bg = background_secondary(light) #FFF1F3F5
- 标题/单位 = font_secondary #99000000；主温度 = font_primary #E5000000
- cold_panel 背板 = comp_background_secondary #19000000（中性背板承载状态）
- 感冒指数值 = alert #FFED6F21（真实健康提示状态，置于中性背板内）
- CTA 背景 = brand #FF0A59F7，前景 font_on_primary #FFFFFFFF（品牌动作实色）

## 校验
- 三行 JSONL：createSurface / updateComponents / updateDataModel。✓
- version v0.9 + extended.catalog；尺寸 2x2 一致。✓
- 所有表达式路径在 updateDataModel.value 可推导。✓
- 仅 Form 组件，无禁用项；onClick 仅用已声明能力。✓
- CardSpec refreshPlan 仅引用已声明 bindingId「shenzhen_weather」。✓
