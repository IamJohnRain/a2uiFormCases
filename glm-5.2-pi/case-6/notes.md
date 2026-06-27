# Case-6 · 大湾区城市马拉松倒计时卡片

## 用户需求

距离大湾区城市马拉松倒计时卡片：偏深色渐变紫色、夜跑氛围；中间突出"距开赛 32 天"，一眼可见；下方显示今日运动计划（8km 配速慢跑）；整体运动感 + 简洁高级。

## 场景与模式

- 场景：`time/reminder` —— 单事件倒计时 + 今日计划。
- 模式：模式 1（一句话桌面卡片，新建，无既有 DSL）。

## 尺寸选择：2x2（160×160vp）

- 内容语义为：identity（赛事名）+ primaryAnswer（倒计时天数）+ context（今日计划）= 3 个主区域，命中 2x2「<= 3 主区域」。
- 用户强调"一眼就能看到"和"简洁高级"。单焦点竖向构图 + 一个巨型 hero 数字，比横版 2x4 更聚焦、更易扫视；3 个小元素放进 2x4 会显得空旷，反而不"高级"。
- 构造规则中"单个事件倒计时用 2x2"；今日计划仅一行短文本，作为底部 context 条带即可装下，无需升级到 2x4。

## 语义角色 → 组件映射

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | 大湾区城市马拉松 | `eventTitle` Text |
| primaryAnswer | 距开赛还有 / 32 / 天 | `focal`(Column) → `countdownCaption` + `countdownRow`(daysNum 46fp 白色 800 + daysUnit 紫色) |
| context | 今日训练 · 8km 配速慢跑 | `planBlock`(半透明白块) → `planTick` 紫色竖线 + `planColumn`(label+value) |

视觉焦点：白色 46fp/800 的"32"，是全卡唯一主导锚点（次大字号仅 16fp，对比 ~2.9×）。

## 视觉语言（深色渐变紫 · 夜跑）

- Root 线性渐变 `RightBottom`：`#150E2C`(夜底) → `#2F1A6B`(紫) → `#6B3AE8`(亮紫光)，深色在左上、亮光在右下，模拟城市夜跑光晕从下方升起。
- 一个连贯紫色色相系统 + 一个亮紫强调家族（planTick `#9B7CFF`、daysUnit `#C9B5FF`、caption `#B3A0E8`）。
- 不使用真实图片（无本地素材）：用渐变 + 半透明块 + 强调 `Divider` 竖线制造"运动感"和"卡片细节感"，避免堆叠多种装饰（expressiveness-toolkit 反模式）。
- 白色"32"压在紫色中部，对比充足；底部 `planBlock` 用 `#22FFFFFF` 白叠层 + `#33FFFFFF` 边框提供层级与可读性。
- Root 柔和阴影（`#20000000`, radius 16, offsetY 6）增加深度。

## 布局预算（2x2, content 132×132vp）

高度（竖向 spaceBetween 分配）：
- eventTitle 12.5fp ≈ 15–16vp
- focal：caption 10fp ≈ 12vp + itemMargin 2 + countdownRow(daysNum 46fp ≈ 55vp) ≈ 69vp
- planBlock：padding 6×2 + (label 9fp ≈ 11 + itemMargin 2 + value 12fp ≈ 15) ≈ 40vp
- 合计 ≈ 125vp，剩 ~7vp 由 spaceBetween 分成两个 ~3.5vp 间隙，留有余量不溢出。

宽度（最紧的两条横向）：
- `countdownRow`：32(≈56vp) + gap 3 + 天(≈16vp) ≈ 75vp，居中于 132vp，充裕。
- `planBlock` 内：132 − padding 12 = 120vp → tick 3 + gap 7 + planColumn 110vp。"8km 配速慢跑"≈71vp、"今日训练"≈36vp 均远小于 110vp。
- 关键信息（32、天、8km 配速慢跑、赛事名、距开赛还有）均无 ellipsis/clip/marquee，无窄固定宽容器，预算内完整显示。

## 交互与 DataModel

- 静态信息卡：用户要求是"随时查看"（扫视），未要求动作；且无已声明宿主函数。故无可点击区域、不发明 `onClick`/宿主函数。
- DataModel 按领域语义分组：`race.{name,caption,daysLeft,unit}`、`plan.{label,value}`；展示字符串预计算，避免脆弱表达式。
- 倒计时天数绑定 `{{ $__dataModel.race.daysLeft }}`（数值 32），`minFontSize:38` 防御三位数加宽。

## CardSpec：静态

- 仅 `{"suggestSize":"2x2"}`，无 `dataBindings`/`refreshPlan`。
- 理由：已声明能力仅有 `calendar.events.search` / `weather.overview.get`；"距开赛还剩几天"需要日期差运算，而 Form 表达式内建函数仅 `size()`，无法表达；用户又给了具体数值（32 天、8km 计划）。按 skill 规则"未声明能力只能降级为静态卡片"，采用静态卡片，不虚构能力/参数/刷新计划。

## 显式改进（首版 → 终版）

首版内部草稿问题：
1. 顶部仅放光秃秃的赛事名，缺乏"运动/夜跑"视觉能量。
2. 今日计划块只是普通文字，缺少"卡片细节/高级感"。
3. 未做高度预算，hero 数字 50fp 时三段内容接近 160vp 溢出。

终版改进：
- 引入竖向紫色强调 `Divider`（planTick）作为 planBlock 的"运动标记"竖线，提升运动感与高级细节感，且不增加横向竞争。
- planBlock 改为半透明白叠层 + 边框 + 圆角，形成 premium 的次级块（外圆角 22 ≥ 内圆角 13）。
- 按高度预算把 hero 数字收敛到 46fp（保留 ~2.9× 主导优势），padding/itemMargin 收紧，确保 132vp 内三段 + 间隙不溢出。
- 渐变选定为深紫夜底 + 亮紫右下光，强化"夜跑"语义而非纯装饰。

## 校验日志

```
$ python3 .../validate_genui_card.py glm-5.2-pi/case-6/card.dsl.jsonl
GenUI 卡片校验通过
消息数：3
组件数：12
surfaceId：marathon-countdown

$ python3 .../validate_cardspec.py glm-5.2-pi/case-6/cardspec.json
CardSpec validation passed
```

两个校验器均 0 error / 0 warning。

## 最终验收（review-validation + design-review）

- 协议阻塞项：version/catalogId/单一 updateComponents/surfaceId 一致/无 theme/仅 Form 组件/Text.content/仅 onClick（本卡无可点击区）/无网络与 SVG 媒体 —— 全部通过。
- 卡片阻塞项：明确 2x2、主区域 3（≤3）、无滚动长列表、规则驱动非模板、有布局理由与改进 —— 通过。
- CardSpec 阻塞项：suggestSize 与 DSL 一致、静态卡不虚构 dataBindings/refreshPlan —— 通过。
- 受保护文本换行：两条横向行均完成宽度预算，关键值无截断、无窄固定宽 —— 通过。
- 视觉：单一主导锚点（32）、一个连贯紫色色相系统、渐变/半透明块/强调线支持场景而非纯装饰、外圆角 ≥ 内圆角、阴影归属 root —— 通过。

## 受限说明

- `accessibility` 标签未添加：权威的 `component-catalog.md` 未在 Form 子集中登记 `accessibility` 属性/样式，按协议优先级（catalog > 设计文档）与"不发明样式键"原则，保持卡片简洁；校验器虽允许顶层非 action 字段，但为不引入未文档化属性而省略，并在 dataBinding 角色上维持语义清晰。这不阻塞交付。
