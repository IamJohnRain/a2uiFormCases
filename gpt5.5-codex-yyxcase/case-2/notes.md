# Case 2 Notes

## Layout Reasoning

- 场景分类：`time/reminder` + `action card`，核心是马拉松倒计时提醒，并支持一键启动今日训练计划。
- 模式：模式 1，新建一句话桌面卡片。
- 尺寸：选择 `2x4`。赛事名、临近天数、今日训练计划和 CTA 都是用户要求显示的关键内容，`160vp` 宽度会让赛事名或 CTA 风险过高；`320 x 160vp` 可以形成左侧倒计时、右侧详情和动作的横向关系。
- 语义角色：identity 是“运动提醒/上海马拉松”；primary answer 是“18 天”；metric 是倒计时和训练距离；context 是日期地点与今日训练摘要；progress/trend 是备赛周期进度环；action 是“启动今日训练”。
- 主区域：左侧倒计时视觉锚点、右侧赛事信息、右侧训练摘要、底部 CTA，共 4 个主区域，符合 `2x4` 限制。
- 视觉焦点：左侧半透明进度环承载 `18 天`，用偏运动户外的深青绿到暖橙渐变表达晨跑和比赛临近。
- 信息节奏：先读倒计时，再读赛事名和日期地点，随后看到今日训练内容，最后执行 CTA。
- 关键横向关系：主 Row 内部为 `countdownPane` 112vp + gap 10vp + `detailPane` 174vp，总计 296vp，匹配 root 内部宽度 `320 - 24 = 296vp`。
- 必须完整显示的关键信息：赛事名、临近天数、赛事日期时间、今日训练标题、CTA 文案均不使用 ellipsis/marquee；通过横版尺寸、`minFontSize` 和独立按钮行保证完整显示。
- 拥挤 Row 宽度预算：
  - root 主 Row：可用 296vp；左 pane 112vp，gap 10vp，右 pane 174vp，合计 296vp。
  - trainingBlock：右 pane 宽 174vp，内部 padding 16vp，accent 3vp，gap 6vp，trainingCopy 可用约 149vp；“今日：节奏跑 8km”和“含热身 · 配速 5'40/km”均为一行短文本，并设置 `minFontSize`。
- 交互和 DataModel：`Button.onClick` 调用宿主假设函数 `openTrainingPlan`，参数从 DataModel 读取 `planId` 与 `eventId`，没有硬编码到事件处理器。
- CardSpec：`suggestSize` 为 `2x4`，与 DSL 尺寸一致。由于 skill 已声明的数据能力只有天气和日历，没有赛事/训练计划能力 manifest，本卡降级为静态 DataModel，不虚构 `dataBindings` 或 `refreshPlan`。

## Explicit Improvement

首个内部版本把状态标签、训练摘要和 CTA 放在右侧同一紧凑区域，可能压缩赛事名和按钮。改进后移除独立状态标签，把状态含义并入“今日：节奏跑 8km”摘要，CTA 单独占一行全宽按钮，保证赛事名、倒计时和“启动今日训练”完整显示。

## Validation Result

- `python .agents/skills/harmony-card-generation-v5/scripts/validate_genui_card.py gpt5.5-codex-yyxcase/case-2/card.genui.jsonl`: passed
- `python .agents/skills/harmony-card-generation-v5/scripts/validate_cardspec.py gpt5.5-codex-yyxcase/case-2/card.cardspec.json`: passed
- 最终验收：passed。按 `review-validation.md` 和 `design-review.md` 检查，未发现协议阻塞项、CardSpec 阻塞项或受保护文本截断问题。

## Host Function Assumptions

- `openTrainingPlan` 是宿主 catalog 已声明的自定义函数，用于打开并启动今日训练计划。
- `openTrainingPlan.args.planId` 接收训练计划 ID。
- `openTrainingPlan.args.eventId` 接收关联赛事 ID，便于宿主进入对应备赛计划上下文。
