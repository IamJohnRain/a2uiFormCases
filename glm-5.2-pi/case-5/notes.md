# case-5 睡眠监测卡片 — 设计与校验笔记

## 用户需求
睡眠监测桌面卡片，深紫色夜晚渐变风格；顶部「睡眠不足」提示；中间突出睡眠时长「6小时45分」并用圆环表现睡眠状态；底部「设定早睡提醒」按钮带闹钟图标；整体安静、柔和、有科技感和高级感。

## 场景识别
`status glance` + 健康/进度。场景映射「健康/进度，单指标」→ 首选 `2x2`。

## 尺寸选择：2x2 / 160×160vp
- 一个主结论（时长/状态）+ 一个上下文（状态徽标）+ 一个动作（提醒）。
- 3 个主区域 ≤ 3，无需滚动、无段落。横版 2x4 没有必要的左右关系。

## 语义角色
| 角色 | 内容 | 组件 |
| --- | --- | --- |
| status | 睡眠不足 | 药丸 Row（● + Text） |
| primaryAnswer | 圆环 + 6小时45分 | Stack(Progress + Column 叠加) |
| action | 设定早睡提醒 | Button（标签含 ⏰ 闹钟字形） |

## 结构与节奏（header-primary-action）
Root Column（深紫渐变 night shell，圆角 22 + clip）
→ statusPill（顶部，半透明白药丸 + 珊瑚点）
→ focalStack（中部，Progress 圆环 86×86 叠加时长）
→ ctaBtn（底部，全宽磨砂浅紫药丸，闹钟字形）

视觉焦点唯一：圆环 + 叠加时长；圆环加紫色辉光阴影增强科技/高级感。

## 布局理由 / 宽度预算
- Root padding 10 → 内容区 140×140vp。
- 状态药丸：●(9fp≈6) + itemMargin5 + 「睡眠不足」(4×11fp≈44) + padding12 ≈ 67vp，居中放得下。
- 圆环 86vp 居中，< 140vp。
- 纵向：padding20 + 药丸≈23 + 圆环86 + 按钮30 = 159 ≤ 160，spaceBetween 微分布。
- CTA 标签「⏰  设定早睡提醒」≈ 字形13 + 双空格8 + 6字×12=72 ≈ 93vp，按钮 width:100%=140vp 完整不挤压。

## 受保护文本完整性
- 「睡眠不足」「6小时」「45分」「设定早睡提醒」均为关键内容。
- 全部 maxLines:1，未使用 ellipsis/clip/marquee。
- 时长拆为「6小时」(16fp 粗) / 「45分」(11fp) 两行叠加，保证圆环内完整且突出。

## 显式改进
首版本会把时长写成单行「6小时45分」塞进 86vp 圆环，会过窄难读。
改进：拆成「6小时」(粗大) + 「45分」(小) 两行叠加 → 既完整又强化时长层级与突出感；
并加圆环紫色辉光阴影 + 磨砂浅紫药丸按钮 + 柔和珊瑚状态点，提升柔和/高级/科技感。

## 视觉与交互
- 调色板（深紫夜晚，一个强调色族 + 一个柔警告色）：
  - 渐变 `#160E36 → #2A1C68 → #4A2BA0`（direction Bottom）
  - 圆环 `#A78BFA` + 辉光 `#666A3FFF`
  - 主文本 `#FFFFFF`，副文本 `#C9BDF5`/`#F1EAFF`
  - 状态点珊瑚 `#FFB3C1`（柔警告，与紫调形成柔和对比）
  - CTA 磨砂浅紫 `#EAE3FF` + 深紫文本 `#33215C`
- CTA 用语义 Button（含 onClick），闹钟图标以字形 ⏰ 置于 label（无真实本地图标素材时按表现力工具箱用字形）。
- 圆环 value/total = 6.75/8（相对 8 小时目标），如实呈现进度；「不足」由珊瑚状态药丸承担判定语义。

## DataModel（语义分组，UI 全表达式绑定）
```
sleep: { status, hoursLabel, minutesLabel, value(6.75), goal(8) }
action:{ label, reminderId }
```

## CardSpec 形态：静态
能力清单（weather / calendar）未声明睡眠/健康 capability → 按规则不虚构 dataBindings。
`suggestSize:"2x2"`，无 dataBindings、无 refreshPlan。

## 宿主假设（需在最终说明）
- CTA `onClick` 调用宿主自定义函数 `setEarlySleepReminder`，参数 `reminderId` 来自 DataModel。
  该函数未在已声明能力清单内，明确标记为宿主假设；端侧需提供 catalog 声明或等价宿主动作。
- 闹钟图标以字形 ⏰ 实现；如端侧有真实本地闹钟图标资源，可替换 label 中的字形或在 Row 内放置独立 Image。

## 校验日志

```
$ python3 scripts/validate_genui_card.py glm-5.2-pi/case-5/card.dsl.jsonl
GenUI 卡片校验通过
消息数：3
组件数：10
surfaceId：sleep-card

$ python3 scripts/validate_cardspec.py glm-5.2-pi/case-5/cardspec.json
CardSpec validation passed
```

两脚本均无错误、无警告。

### 最终验收（review-validation + design-review）
- 协议阻塞项：version v0.9 / catalogId / 无 theme / 单次 updateComponents / surfaceId 一致 / 引用可解析 / 仅 Form 10 组件 / extended 属性名 / 仅 onClick — 全部通过。
- 卡片阻塞项：已选 2x2、主区域 3≤3、无需滚动、规则驱动非模板、已有布局理由与显式改进、CTA 有真实 onClick — 全部满足。
- CardSpec 阻塞项：suggestSize 与 DSL 一致；静态卡未虚构 dataBindings/refreshPlan — 满足。
- 受保护文本：「睡眠不足」「6小时」「45分」「设定早睡提醒」无 ellipsis/clip/marquee，完整显示。
- 设计评审：单视觉锚点（圆环+时长）；外圆角 22 ≥ 内圆角(药丸12/按钮15)；阴影集中于 root+圆环辉光+CTA；无网络/占位媒体；CTA 参数从 DataModel 绑定；关键区域有 accessibility label。无需返工。
