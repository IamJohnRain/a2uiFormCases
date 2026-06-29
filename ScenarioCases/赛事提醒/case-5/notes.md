# case-5 大湾区城市马拉松倒计时卡片

## 需求要点
- 深色渐变紫、夜跑氛围
- 中部突出「距比赛还有 32 天」，一眼可见
- 下方显示今日运动计划（8km 配速慢跑）
- 运动氛围、简洁高级；尺寸 2x2

## 布局理由（big-number-support 原型）
内容区 116×116（root padding 12）。竖向三段，组间距 8vp：
- header 16vp：赛事名 + 紫点标识
- hero 48vp：40fp「32」绝对主数值 + 12fp「天后开跑」标签（底对齐）
- plan 34vp：半透明背板，左侧紫色竖条 + 「今日计划」标签 + 「8km 配速慢跑」内容

竖向预算：16 + 8 + 48 + 8 + 34 = 114 ≤ 116 ✓
- header 横向：dot6 + gap6 + raceName104 = 116 ✓
- hero 横向：「32」≈48 + gap4 + 「天后开跑」48 = 100 ≤ 116 ✓
- plan 内容区 = 116 − padding16 = 100；icon4 + gap8 + planCol80 = 92 ≤ 100 ✓
- planText「8km 配速慢跑」≈ 22 + 4 + 48 = 74 ≤ 80 ✓ 完整显示

## 颜色来源（场景拓展色，temporal-band 倒计时 + fitness 夜跑紫）
- scene: `event.countdown.night` / `fitness.night`
- anchors: 夜跑、城市夜灯、跑道、紫调马拉松视觉
- baseSources: `multi_color_01`(#5f58c7 dark)、`multi_color_06`(#8c55c2 dark)、`multi_color_aux_06`(#634794 dark)
- derivedRoles:
  - rootStart `#1B1038` / rootEnd `#3A1F6E`：基于 01/06/aux06 暗化的夜紫渐变（线性 150°，2 stop，倒计时语义）
  - accent `#A77DFF`：multi_color_06 提亮的强调紫，用于标识点、"天后开跑"、计划竖条
  - softPanel `#1FFFFFFF`：comp_background_secondary(dark #19FFFFFF) 同源弱背板
- 文本：font_on_primary #FFFFFFFF（hero）、#E5FFFFFF/#CCFFFFFF/#99FFFFFF（on-color 中性前景层级）
- 渐变背景上仅一个强调彩色（accent 紫），无第二高饱和前景 ✓

## 信息职责互斥
- raceName=对象；hero=倒计时主答案；planTag=标签角色「今日计划」；planText=计划内容
- 「今日」由 planTag 承担，planText 不再重复，无事实等价类重复 ✓

## 校验
- L0 协议：三行 JSONL / v0.9 / extended.catalog / root 引用存在 / 仅用 Column·Row·Stack·Text ✓
- 尺寸：2x2 140×140 borderRadius18 clip，CardSpec 一致 ✓
- 绑定：race.name / race.daysLeft / plan.todayLabel 均在 updateDataModel ✓
- 静态卡片，未虚构 dataBindings / refreshPlan ✓
- 无网络图/SVG/emoji/禁用组件/事件能力 ✓
