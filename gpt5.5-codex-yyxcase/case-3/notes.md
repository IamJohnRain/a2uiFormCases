# Case 3 睡眠监测卡片

## 布局理由

- 场景/模式：新卡片，`status glance` + `time/reminder` + 单一 `action card`，使用模式 1。
- 尺寸：选择 `2x4`。睡眠时长、目标进度、睡眠质量、早睡提醒和 CTA 都是用户关键信息，`2x2` 会让主指标和提醒动作争抢空间；横版 320 x 160vp 能保持一眼速览。
- 语义角色：`identity` 是“睡眠监测”，`primaryAnswer` 是 `7h 35m`，`metric` 是进度条和完成 95%，`context` 是目标/深睡/早睡时间，`action` 是“设定早睡提醒”。
- 主区域：左侧主指标块、右侧详情块、中部进度条、底部提醒动作，共 4 个主区域，符合 `2x4` 上限。
- 视觉焦点：左侧大号睡眠时长 + 中部线性 Progress。深夜渐变和半透明块表达睡眠场景，不使用未提供的图片资源。
- 信息节奏：先读标题与时长，再扫目标/质量/早睡时间，然后看进度条，最后执行提醒设置。
- 关键横向关系：左侧主指标与右侧详情并排；进度元信息左右对齐；底部弱提示让位给固定宽度 CTA。
- 必须完整显示的关键信息：睡眠时长、完成百分比、目标时长、早睡时间、CTA“设定早睡提醒”。这些 Text/Button 未使用 ellipsis；只有底部建议文案是可压缩弱文本。
- 拥挤 Row 宽度预算：
  - `topRow` 可用宽度约 296vp，左块 118vp + gap 12vp，右块约 166vp，详情行每行仅“标签 + 短值”，可完整显示。
  - `bottomRow` 可用宽度约 296vp，CTA 固定 104vp + gap 8vp，提示文本约 184vp；提示为弱文本可 ellipsis，CTA 完整显示。
  - `progressMeta` 可用宽度约 296vp，左右短文本“完成 95%”和“目标 8h”合计远小于可用宽度。
- 交互和 DataModel：DataModel 按 `sleep`、`reminder`、`action` 语义分组；Button 使用 `onClick` 调宿主函数 `openSleepReminderSettings`，参数来自 DataModel。
- CardSpec：`suggestSize` 为 `2x4`。当前参考只声明天气/日历能力，没有睡眠监测或提醒持久化能力，因此不虚构 `dataBindings`，以静态初始化数据交付；真实睡眠刷新需要端侧补充 capability manifest。

## 一次显式改进

内部首版把 CTA 做成右侧窄动作栏，但“设定早睡提醒”是受保护文本，在 44-60vp 右栏内容易被截断。已改为底部 104vp 宽按钮，并将提示文案改成可压缩文本，保证 CTA 完整显示。

## 校验结果

- `python .agents\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-codex-yyxcase\case-3\card.genui.jsonl`：通过。消息数 3，组件数 26，surfaceId 为 `sleep-monitor-card`。
- `python .agents\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-codex-yyxcase\case-3\card.cardspec.json`：通过。
- 最终验收：已按 `review-validation.md` 进入并读取 `design-review.md`。尺寸、主区域数量、受保护文本、CardSpec 对齐、交互事件和静态能力降级均通过检查。

## 宿主函数假设

- `openSleepReminderSettings`：宿主 catalog 已声明的自定义函数，用于打开早睡提醒设置；接收 `reminderId` 和 `defaultTime`。
