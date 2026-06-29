# 深圳天气卡片 — case-1 notes

## 场景与取舍
- 动态卡片，使用 `weather.overview.get` 能力，`prefectureName: 深圳市`，`forecastDays: 1`。
- 主答案：当前温度（40fp 单一主数值）。
- 支撑事实（2 条）：最低/最高温区间；空气质量。
- 城市名 + 天气状况作为标题行的对象/状态承载。
- 主动作：整卡 `onClick` 跳转华为天气 App（clickToDeeplink，目标在 click-event manifest 内）。

## 尺寸与布局预算（2x2）
- surface/root 一致：width=140, height=140, padding=12, borderRadius=18, clip=true。
- 内容区 116x116，root 为 Column，justifyContent=spaceBetween。
- 三个主区域（topRow / tempRow / bottomRow），符合 2x2 ≤3 主区域 + 1 动作。
- topRow: 116x18，城市(14fp)左、状况(12fp)右，spaceBetween。
- tempRow: 116x46，温度 40fp + 单位 16fp（margin.top 6 做基线对齐），宽度充足。
- bottomRow: 116x16，区间 12fp 左、空气 12fp 右，贴底安全（padding 12 即底边距 12vp，在 8-16 区间内）。
- 受保护文本：城市、温度、区间均 maxLines:1 + textOverflow:none，无截断。

## 颜色来源
- root 线性渐变天气氛围带（ambient/temporal-band，weather 场景允许）。
- 场景拓展色 scene=weather.day.cool，anchors=晴空/海湾天空，baseSources=multi_color_02(#46b1e3)/multi_color_08(#0a59f7) 蓝色族，
  derivedRoles: rootStart=#FF4A90D9, rootEnd=#FF7FB5E6（低饱和、高明度、同一冷蓝族，2x2 ≤2 拓展色）。
- 渐变上文字用 on-color：主文字 #FFFFFFFF (font_on_primary)，次级 #E5FFFFFF。
- 单一主场景色族（冷蓝），无第二高饱和前景。

## 数据绑定
- 展示值优先完整表达式 `{{ $__dataModel.data.weather.items.weatherData.* }}`。
- 路径由 writeResultTo(/data/weather) + weather outputSchema 推导：
  - cityInfo.cityName / weatherData.weather_icon / temperature / low_temp / high_temp / aqivalue。
- 区间与空气文案用单字符串内一对 {{ }} 拼接，符合表达式完整性。
- 初始 DataModel 用空对象/空数组 + state.loading，不写死真实天气。

## 校验
- 模型内置校验通过：L0 协议（三行 JSONL、catalog、尺寸一致、root shell、Form 组件子集）、
  L1 布局预算（并排宽度求和不超父宽、贴底）、颜色来源可回溯、事件能力合法（clickToDeeplink/天气 App）、
  尺寸 2x2 与 cardspec 一致。
- 静态草稿未运行 validate 脚本（按规则仅用户要求时运行）。
