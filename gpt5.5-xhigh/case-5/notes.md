# 布局理由与校验记录

## 场景与尺寸

- 场景识别：睡眠/健康状态速览，属于 status glance，带一个提醒动作。
- 模式：一句话桌面卡片。
- 尺寸选择：`2x2`，因为核心信息是一个状态、一个主睡眠时长、一个 CTA，`160 x 160vp` 可无滚动容纳。

## 语义角色

- identity/status：`睡眠不足` 顶部提示。
- primaryAnswer/metric：`6小时45分` 是主指标。
- progress：`Progress` 使用 `ring` 类型，以 `405 / 480` 表示相对 8 小时目标的睡眠状态，作为协议内的圆环抽象。
- action：底部 `⏰ 设定早睡提醒` 按钮。

## 主区域与信息节奏

- 主区域 1：顶部状态提示，约 22vp 高。
- 主区域 2：中部圆环和时长，约 78vp 高，是视觉焦点。
- 主区域 3：底部 CTA，30vp 高。
- 信息节奏：状态先告知风险，中间给出大指标，底部给出立即设置提醒的下一步。

## 视觉焦点

- 深紫夜晚渐变作为 root shell，两个半透明光晕营造安静、柔和的科技感。
- 没有使用图片资源；按协议用渐变、Stack、Progress、文字字形表达夜晚氛围和圆环。

## 横向关系与宽度预算

- Root `160vp`，padding 不额外占用，内容层宽 `140vp`。
- 顶部 Row 宽 `136vp`，两个子项：状态 badge 约 `74vp`，月亮标记约 `15vp`，itemMargin `6vp`，剩余空间约 `41vp`，受保护状态短语可完整显示。
- CTA Button 宽 `136vp`，标签 `⏰ 设定早睡提醒` 使用 `12fp` 和 `minFontSize: 10`，不使用 ellipsis/clip，确保关键 CTA 完整显示。
- 中部圆环 `78vp`，内部文本列 `66vp`，`6小时45分` 用 `17fp` 和 `minFontSize: 14`，可完整显示。

## DataModel 与 CardSpec

- DataModel 按语义分组为 `sleep` 和 `action`，展示字符串预计算，避免复杂表达式。
- CardSpec 为静态卡片，只声明 `suggestSize: "2x2"`，不虚构未声明的睡眠数据能力或刷新计划。

## 显式改进

- 初版设想把状态、目标和 CTA 挤在多个横向小标签里，容易使 `睡眠不足` 和 CTA 受保护文本被挤压。
- 改进后保留 3 个主区域，目标说明放入圆环中心的小字，CTA 独占底部全宽按钮，减少横向拥挤。

## 宿主假设

- CTA `onClick` 使用宿主 catalog 已声明的自定义函数假设：`setEarlySleepReminder`。
- 假设该函数接收 `reminderId` 和 `recommendedTime`，用于打开或创建早睡提醒设置。

## 校验记录

- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-xhigh\case-5\genui.jsonl`：通过。
- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-xhigh\case-5\cardspec.json`：通过。
