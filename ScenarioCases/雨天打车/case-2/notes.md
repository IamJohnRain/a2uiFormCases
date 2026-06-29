# 雨天打车 case-2 设计与校验记录

## 场景与主答案
- 场景：雨天叫车小组件（2x2，静态卡片）。
- 主答案：雨天出行 → 建议打车。围绕一个主对象（雨天出行决策）展开，不重复同一事实。
- 信息职责（互斥）：
  - header = 天气情景标签（对象/状态）
  - hero = 出行建议（主答案，配车图标做识别）
  - tip = 安全提示（一条支撑事实）
  - cta = 动作入口（查看雨势）

## 布局（top-hero + 动作，Column 纵向流）
- root 内容区 116×116（root padding 12）。
- 纵向预算：header 22 + hero 30 + tip 16 + cta 32 = 100；itemMargin 4 × 3 间隔 = 12；合计 112 ≤ 116。通过。
- header 横向：图标 20 + gap 6 + 标题文本（“雨天出行提醒”6字×14fp≈84）= 110 ≤ 116。通过。
- hero 横向：车图标 26 + gap 6 + “建议打车”（4字×18fp≈72）= 104 ≤ 116。通过。
- tip：“路面湿滑，注意安全”8字×12fp≈96 + 标点，≤116，单行不截断。
- CTA 与上方内容同宽 116，height 32 ≥ 24vp 热区，贴近底部安全区（spaceBetween + 末位），底边距约 12vp。

## 颜色来源
- root 渐变：基于 `background_secondary`(#FFF1F3F5)/`background_tertiary` 派生的低饱和蓝灰玻璃表面（雨天场景拓展色 `weather.rain.cool`，锚点：雨幕/湿玻璃），stop `#FFE9EEF3`→`#FFD4DCE5`，仅做线性氛围带，不挤压前景。
- 文本：`font_primary` #E5000000、`font_secondary` #99000000，普通背景配对合规。
- CTA：`brand` #FF0A59F7 实色 + `font_on_primary` #FFFFFFFF，仅用于唯一主动作。
- 单一主场景色族（蓝灰中性），无第二高饱和前景。

## 事件
- onClick = clickToDeeplink → 天气应用首页（manifest supportedTargets 合法目标）。
- 说明：事件能力 manifest 中无打车/网约车应用目标，故 CTA 落地为“查看雨势”跳转天气，避免“一键叫车”却跳转天气的语义错配。

## 协议/契约校验
- genui 为三行 JSONL：createSurface / updateComponents / updateDataModel；version v0.9，catalogId extended.catalog。
- 尺寸一致：DSL 140×140、root borderRadius 18、clip true；cardspec suggestSize 2x2。
- 仅使用 Form 组件：Column/Row/Image/Text/Button。
- 所有可见表达式（weather/suggest/tip/cta）均在 updateDataModel.value 中存在。
- 静态卡片：cardspec 只含 suggestSize，未虚构 dataBindings/refreshPlan。
- 素材均来自素材库 taxi_widget（icon_weathe2.png、icon_car.png），本地资源路径，无网络图/SVG/emoji。

## 取舍
- 未引入“预计接驾分钟数 / 车费”等动态数值：当前无打车 data-capability manifest，避免虚构动态字段；改为静态决策提示卡，保证可上线、可渲染。
