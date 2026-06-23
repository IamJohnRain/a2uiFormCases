# Case 7 Notes

## Layout Reasoning

- 场景分类：time/reminder + action card，属于会议/日程提醒能力范围。
- 模式：模式 1，新建一句话桌面卡片。
- 尺寸：选择 `2x2`，因为用户需求是单个下一个日程、一个主时间、一个 CTA，能在 `160 x 160vp` 内无滚动容纳。
- 语义角色：identity = `下一个日程` 与 `Agent需求评审会`；primaryAnswer/metric = `14:00-15:30`；context/status = `还有15分钟后开启`；action = `☾ 专注模式`。
- 主区域：顶部身份信息、中间半透明主时间块、底部 CTA，共 3 个主区域。
- 视觉焦点：中部大号会议时间，暖咖啡渐变背景与柔和光晕服务于焦点，不使用图片。
- 信息节奏：小标题 -> 会议名 -> 大时间 -> 倒计时 -> 专注 CTA。
- 关键横向关系：倒计时行仅有圆点和倒计时文本；CTA 独占整行按钮，避免挤压。
- 必须完整显示的关键信息：`下一个日程`、`Agent需求评审会`、`14:00-15:30`、`还有15分钟后开启`、`☾ 专注模式`。
- 拥挤 Row 宽度预算：`countdownPill` 位于主块内，父级约 `138vp`，主块 padding `18vp` 后可用约 `120vp`，圆点约 `8vp` + margin `4vp`，倒计时文本可用约 `108vp`，11fp 的 `还有15分钟后开启` 可完整显示。
- CTA 宽度预算：按钮使用内容宽 `138vp`，`☾ 专注模式` 13fp 可完整显示。
- 交互和 DataModel：所有可见业务文本来自 `/meeting` 和 `/action`；CTA `onClick` 参数从 DataModel 绑定会议 ID 和模式。
- CardSpec：静态卡片，`suggestSize` 为 `2x2`，不虚构 `dataBindings` 或 `refreshPlan`。

## Explicit Improvement

- 初版内部构想把 `会议名`、`时间`、`倒计时` 都放在同一个半透明块内，顶部只留标签；这样会议名和时间层级竞争且中部过挤。
- 改进后将会议名放回顶部身份区，中间半透明块只承载大时间和倒计时，底部 CTA 独占整行，提高主时间可读性并保证按钮完整显示。

## Host Assumption

- CTA `onClick` 使用宿主自定义函数假设：`enableDoNotDisturb` 已在宿主 catalog 中声明，用于根据 `meetingId` 和 `mode` 快速开启勿扰/专注模式。

## Validation Log

- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-xhigh\case-7\genui.jsonl`：通过。消息数 3，组件数 12，surfaceId `meeting-reminder-focus-card`。
- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-xhigh\case-7\cardspec.json`：通过。

## Final Review

- 协议：使用 `v0.9`、extended catalog、一次完整 `updateComponents`、扁平组件邻接表、Form 子集组件。
- 尺寸：DSL root 为 `160 x 160vp`，CardSpec `suggestSize` 为 `2x2`，一致。
- 受保护文本：关键标题、时间、倒计时、CTA 均未使用 ellipsis/clip/marquee，且通过拆区与整行按钮保证完整显示。
- 数据：静态 DataModel 可推导所有 UI 绑定路径；未声明端侧动态能力，因此 CardSpec 不包含 `dataBindings`。
