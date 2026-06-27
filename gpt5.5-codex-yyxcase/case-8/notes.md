# case-8 抖音使用时长检测卡片

需求：显示时长设定以及已超时间，附带进度条可视化，支持设定管控时间。

## 布局理由

- 尺寸选择：`2x4`。需要同时展示设定时长、已超时间、进度可视化和管控动作，横版能保护数字和 CTA。
- 语义角色：`identity` 为抖音时长，`primaryAnswer` 为已超时间，`metric` 为设定/已用/百分比，`status` 为超时，`action` 为设定管控。
- 主区域：左侧应用与环形进度，中部超时详情和线性进度，右侧状态与按钮。
- 视觉焦点：环形进度与大号“18 分钟”共同构成焦点，线性进度补充比例。
- 宽度预算：root 内宽约 `296vp`，左 pane `116vp`，右栏 `60vp`，间距 `20vp`，中部约 `100vp`；受保护文本为 `60 分钟`、`18 分钟`、`130%`、`设定管控`，均不使用省略。
- 交互：`Button.onClick` 调用宿主函数 `setAppControlTime`，参数从 DataModel 绑定。
- CardSpec：未提供屏幕使用时长能力 manifest，按静态卡片处理。

## 显式改进

首版把“设定 60 分钟”和“已超 18 分钟”放在同一横向行，数字竞争空间。改进后将设定值放在左 pane，已超时间作为中部主指标，保证关键字段完整。

## 校验日志

- `validate_genui_card.py`：通过，消息数 3，组件数 18，surfaceId 为 `case-8-douyin-usage`。
- `validate_cardspec.py`：通过。

## 宿主函数假设

- `setAppControlTime(appId, profileId)`：宿主 catalog 已声明，用于打开或设置应用管控时间。
