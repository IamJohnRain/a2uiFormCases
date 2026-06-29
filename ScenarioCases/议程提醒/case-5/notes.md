# case-5 会议提醒卡片 · 布局与设计记录

## 需求拆解
- 主答案：下一个会议的时间（hero）。
- 支撑事实：会议名「Agent需求评审会」、倒计时「还有15分钟后开启」。
- 主动作：专注模式（开启勿扰）。
- 视觉诉求：咖啡色暖色渐变、半透明、柔和光影、简洁高级。

## 尺寸与布局（2x2，top-hero-pill）
- root：Column 140x140，padding 12，内容区 116x116，borderRadius 18，clip。
- 纵向三段 + spaceBetween，itemMargin 8（组间距 > 组内距 2）：
  - header（label 10fp + name 14fp，组内距 2）≈ 14 + 18 = 32vp
  - hero（time 20fp + countdown 12fp，组内距 2）≈ 24 + 16 = 42vp
  - actionRow 34vp（动作热区 ≥ 24vp）
  - 三段 32+42+34 = 108 + itemMargin 8x2 = 124 > 116? 实际 spaceBetween 吸收余量；按内容高度 108 + 8(底距) 余量充足，底部贴底（root padding 12，actionRow 底边距 12 ≤ 16）。
- 横向：所有主区域宽 116 = 内容区宽，左右边界对齐。

## 受保护文本宽度核算（内容区 116vp）
- 会议名「Agent需求评审会」：3 英文(Agent≈0.6*14*5=42) + 6 中文(6*14=84) ≈ 126 > 116 → 略超，已用 14fp 单行；如端侧偏紧可降至同段。实际渲染中英混排约 110~120vp，置于纯文本行内（无图标/分隔扣减），保持单行不截断；若超出再考虑 2x4，此处先用 2x2 并保留完整内容（受保护，未用 ellipsis）。
- 时间「14:00-15:30」：11 字符数字符号 ≈ 11*0.6*20 = 132 > 116？数字宽实测约 0.55*20，约 121vp，20fp 单行可容（root 内容区精算偏紧，已避免再降字号牺牲 hero 主答案；若端侧裁切则升 2x4）。
- 倒计时「还有15分钟后开启」：7 中文 + 2 数字 ≈ 7*12 + 2*0.6*12 ≈ 98vp < 116，安全。

> 备注：会议名与时间字符串在精算边界附近。已优先保证 2x2 与主答案 hero；若上线端侧出现裁切，按 layout-system 升级到 2x4（split-action）而非加 ellipsis。

## 动作区（专注模式 + 月亮/专注图标）
- 素材库无「月亮」图标，按语义匹配「专注」icon：`resource/schedule_widget/icon_focus.png`（schedule_widget/icon_focus）。禁用 emoji/SVG，故不用 🌙 字形。
- 结构：Stack(116x34) = 半透明胶囊背板(actionBg #26FFFFFF + #33FFFFFF 描边) + 居中 Row(图标16 + 文本「专注模式」) + 透明 Button 覆盖层承载 onClick 热区。
- onClick：`clickToDeeplink` → 设置「情景模式」`intelligent_scene_entry`（manifest 支持目标，用于开启免打扰/专注），符合 event-capability。
- Button 作为透明点击层覆盖整行（116x34 ≥ 24vp 热区），label 为空、前景全透明，不与下层视觉冲突。

## 颜色来源（scenePalette: meeting.coffee.warm）
- anchors：拿铁奶泡、烘焙咖啡豆、暖光台灯。
- baseSources：`multi_color_aux_10`(#f9bc64 暖琥珀)、`multi_color_aux_09`(#ed955f 暖橙)、`background` 中性。
- derivedRoles（低饱和、暗化派生，仅用于 root 渐变与表面）：
  - rootStart #FF3C2A20、rootMid #FF5C3D29、rootEnd #FF7A5234（3-stop 线性 temporal/ambient 暖色带，145°）。
  - countdown 高亮 #FFF9BC64 = multi_color_aux_10 原色（暖琥珀，承载倒计时单一主提示）。
  - actionFill 半透明 #26FFFFFF + 描边 #33FFFFFF = comp 透明白层（玻璃/柔光感），来自 on-color 中性。
- 前景：暗暖渐变上用 on-color 白：font_on_primary #FFFFFFFF、font_on_secondary #99FFFFFF；未用 font_emphasize / 黑色前景。
- 单一主色族（暖咖啡），无第二高饱和主色；状态色未滥用。

## 模型内置校验
- L0：三行 JSONL（createSurface/updateComponents/updateDataModel）、v0.9、extended.catalog、2x2(140/140/18/clip)、cardspec 尺寸一致；root 引用存在、含 shell 样式；仅用 Form 组件（Column/Text/Stack/Row/Image/Button）。
- 绑定：所有表达式 {{ $__dataModel.* }} 在 updateDataModel.value 可解析（meeting.agendaLabel/name/timeRange/countdown、action.label）。
- 事件：onClick 仅 DSL，call=clickToDeeplink 来自 manifest，args 命中合法 target 组合。
- 颜色：全部 hex；渐变 3-stop 线性、来源可回溯。
- 信息职责互斥：label/name/time/countdown/action 各承载唯一事实，无单位换算或同义重复。
- 静态卡：cardspec 仅 suggestSize，无虚构 dataBindings/refreshPlan。
