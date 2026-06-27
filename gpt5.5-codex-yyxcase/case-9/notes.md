# case-9 电量提醒卡片

需求：显示电量以及小字提醒，支持切换省电模式。

## 布局理由

- 尺寸选择：`2x2`。这是单一设备电量状态 + 一条提醒 + 一个动作，可在 160vp 内完整展示。
- 语义角色：`identity` 为电量提醒，`primaryAnswer` 为 28%，`metric` 为预计可用时间，`context` 为小字提醒，`action` 为切换省电模式。
- 主区域：顶部标题状态、中心电量环和百分比、底部提醒与按钮，共 3 个主区域。
- 视觉焦点：环形 `Progress` 和大号电量百分比。
- 宽度预算：root 内宽约 `138vp`；中心 Row 中进度环 `58vp`、间距 `8vp`，文本列约 `72vp`，`28%` 和预计时间均能完整显示；CTA 使用全宽按钮。
- 交互：`Button.onClick` 调用宿主函数 `togglePowerSaveMode`，参数从 DataModel 绑定。
- CardSpec：未提供电池端侧能力 manifest，按静态卡片处理。

## 显式改进

首版将小字提醒和按钮放在同一行，CTA 太窄。改进后提醒独立成一行，按钮全宽，保证“切换省电模式”完整显示。

## 校验日志

- `validate_genui_card.py`：通过，消息数 3，组件数 11，surfaceId 为 `case-9-battery-reminder`。
- `validate_cardspec.py`：通过。

## 宿主函数假设

- `togglePowerSaveMode(modeId)`：宿主 catalog 已声明，用于切换省电模式。
