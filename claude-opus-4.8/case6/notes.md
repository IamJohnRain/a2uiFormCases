# Case6 大湾区城市马拉松倒计时卡片 — 生成记录

## 用户需求
深色渐变紫色夜跑风格的马拉松倒计时卡片：中间突出“距离比赛还有 32 天”，下方显示今日运动计划（8km 配速慢跑），整体运动氛围、简洁高级。

## 场景 / 模式 / 尺寸
- 场景：`time/reminder`（倒计时）+ 次要 `status glance`（今日训练计划）。对应 capability.md 的“倒计时/训练计划”场景。
- 模式：模式 1（一句话生成新卡片，用户未提供已有 DSL）。
- 尺寸：**2x4 / 320x160vp**。依据 card-composition-rules：“倒计时 — 单个事件用 2x2，事件 + 小计划用 2x4”。本需求同时要倒计时 + 今日计划 + 动作，2x4 给受保护文本更安全的横向空间。
- 形态：**静态卡片**。没有匹配的已声明端侧能力（清单只有 calendar/weather，二者均不适用马拉松倒计时与跑步计划），因此不虚构 dataBindings，CardSpec 仅含 `suggestSize`。

## 语义角色
- identity：`race.name` 大湾区城市马拉松。
- primaryAnswer（主视觉焦点）：`race.days` = 32，48fp 大号，配 `daysUnit` 天，baseline 对齐成一个整体。
- context：`race.caption` 距离比赛还有；右侧 `plan`（8 km 慢跑 + 配速 6'00"/km）。
- action：`action` 开始今日训练（Button + onClick）。

## 布局理由
- Root：`Stack`（320x160，borderRadius 22，clip）。三层：
  1. `bgBase` 深色夜跑渐变 `#15121F → #3A2A6E → #6E4BE0`（RightBottom）。
  2. `glow` 半透明紫色光晕块 `#338C6BFF`，制造夜跑氛围（唯一次级视觉装置）。
  3. `content` Row：左 focal pane / 竖直 Divider / 右 detail pane，padding 14。
- 横向节奏：`focal-detail-action`。两 pane 均 `layoutWeight:1`，内容宽 ≈ 320-28(padding)-1(divider)-gaps ≈ 每 pane ~135vp，受保护文本充裕。
- 左 focal（Column 居中）：raceName(12fp,1行) → countRow[32(48fp) + 天(15fp)，alignItems bottom] → caption(10fp)。
- 右 detail（Column）：planLabel(10fp) → planBlock（唯一主半透明块 `#24FFFFFF`+边框，内含左强调 Divider + planTitle “8 km 慢跑” 16fp + planPace 11fp）→ actionBtn 全宽 pill。
- 交互：actionBtn.onClick → `startTrainingPlan(planId)`，planId 由 DataModel 绑定。`startTrainingPlan` 标记为**宿主假设自定义函数**。
- DataModel 按语义分组：`race` / `plan` / `action`，展示字符串预先计算，便于本地化与校验。

## 受保护文本宽度预算
- countRow：32 @48fp ≈ 38vp + gap 4 + 天 @15fp ≈ 16vp ≈ 58vp，远小于 focal pane ~135vp。完整显示。
- planBlock：accent(3vp) + gap8 + planText(layoutWeight:1)。planTitle/planPace maxLines:1，pane 内可用 ~115vp，8 km 慢跑、配速 6'00"/km 完整显示。
- Button 标签“开始今日训练”@12fp ≈ 72vp < pane ~135vp，完整显示。
- 所有受保护文本 `textOverflow:"none"`，无 ellipsis/clip/marquee。

## 显式改进（v1 → v2）
v1 把倒计时数字与单位竖向堆叠、右侧计划用纯文本平铺，导致“32”锚定感弱、右栏扁平。
改进：
1. “32”与“天”改为 baseline 对齐的 Row，使大数字成为单一主导焦点。
2. 计划包进带左强调 Divider 的单个半透明块，给右栏结构与夜跑层次，且不引入第二个竞争 hero（2x4 半透明块上限 2，仅用 1）。
3. 数字后加紫色光晕块强化“夜跑”氛围，作为唯一次级视觉装置。

## 校验日志
GenUI（scripts/validate_genui_card.py，对 card.genui）：
```
GenUI 卡片校验通过
消息数：3
组件数：21
surfaceId：gba-marathon-countdown
```
（初版含一个未引用的孤儿组件 planMeta，已删除后重校，0 错误 0 警告。）

CardSpec（scripts/validate_cardspec.py，对 card.cardspec.json）：
```
CardSpec validation passed
```

## 最终验收结论
- 协议阻塞项：全部通过（v0.9、extended catalog、单次 updateComponents、surfaceId 一致、root 存在、引用可解析、仅用 Form 组件、Text.content/Button.label/onClick 合规、无网络/SVG 媒体）。
- 卡片阻塞项：明确选 2x4；主区域 3 个（≤4）；单一主导焦点；无滚动/长列表/段落；规则构造非模板；有布局理由+显式改进；动作有 onClick。
- CardSpec 阻塞项：suggestSize 2x4 与 DSL 一致；静态卡片未虚构 dataBindings/refreshPlan；UI 路径均可从初始 DataModel 推导。
- 受保护文本：无截断风险。
- 宿主假设：自定义函数 `startTrainingPlan`。
- 结论：通过，可交付。
