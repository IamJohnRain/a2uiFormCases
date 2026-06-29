# 睡眠监督卡片 case-4 设计说明

## 需求映射
- 顶部：睡眠图标 + 标题（当前睡眠状态）。
- 中部圆环：Progress ring，value=已睡时长，total=8，表示占 8 小时比例。
- 右侧：具体睡眠时长（hoursLabel "5.5h"）+ 目标提示。
- 不足 7 小时标题标红：`fontColor` 用条件表达式 `hours < 7 ? '#ffe84026'(warning) : '#e5000000'(font_primary)`。
- 底部按钮：clickToDeeplink → 闹钟 app（com.huawei.hmos.clock / com.huawei.hmos.clock.phone / uri ""）。

## 尺寸 / 布局预算（2x2，140x140，padding 12 → 内容 116x116）
纵向（Column spaceBetween，itemMargin 6）：
- topRow 20 + heroRow 58 + alarmBtn 28 = 106，三段 + spaceBetween 余 10vp 分配为组间留白，<116 成立。
- 底部按钮贴底，root padding 12 即底边距，符合 2x2 底部 8-16vp。

横向 topRow（116）：icon 18 + gap 6 + title 92 = 116 成立。
横向 heroRow（116）：ring 56 + gap 10 + hoursCol 50 = 116 成立。
- hoursValue "5.5h" 20fp：数字/字母约 0.6*20=12，≈ 4 字符 < 50 宽，完整显示。
- statusTitle 14fp，92 宽 ≈ 6.5 个中文字，"睡眠不足，建议早睡" 8 字偏紧 → 实际首选短文案，若超长可降为 "睡眠不足"。

## 颜色来源
- root bg `#fff1f3f5` = background_secondary(light) / 中性表面。
- ring `#564af7` = multi_color_01（睡眠夜间柔紫，scene sleep.night 锚点）。
- 标题红 `#ffe84026` = warning(light)，仅在 hours<7 真实状态触发。
- 正文 `#e5000000` font_primary、`#99000000` font_secondary。
- 按钮实色 `#564af7` 主场景色族 + `#ffffffff` font_on_primary，单一强动作允许实色 CTA。

## 表达式优先
- 标题、圆环 value、时长标签均用完整表达式 `{{ $__dataModel.* }}`。
- 圆环 total=8、目标提示为静态字面量。
- 条件红字用三元表达式（单对 {{ }}，无嵌套）。

## 校验
- L0：三行 JSONL，version/catalogId/尺寸一致，root 引用存在组件且含 width/height/padding/borderRadius/clip/backgroundColor。组件均在 Form 子集内。
- 绑定：sleep.statusTitle / hours / hoursLabel 均在 updateDataModel.value 初始化。
- 事件：clickToDeeplink + 闹钟 supportedTargets 合法组合。
- 信息职责互斥：状态/比例(环)/具体值/目标 各一处，无重复。
