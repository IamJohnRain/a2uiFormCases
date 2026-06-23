# 父母关怀卡片生成记录

## 布局理由

- 场景/模式：家庭关怀 + 天气状态 + 明确拨号动作，属于 status glance 与 action card 的组合，使用模式 1 新卡片。
- 尺寸：选择 `2x4` / `320 x 160vp`。原因是同时需要展示深圳温度、感冒指数、关怀提示和一键打电话；`2x2` 会挤压受保护温度、指数和 CTA。
- 语义角色：identity 为 `深圳` 与 `爸妈天气关怀`；primary answer 为温度；metric 为温度和感冒指数；context 为关怀提示与刷新说明；action 为 `打电话`。
- 主区域：左侧天气焦点、中间关怀详情、右侧拨号动作，共 3 个主区域，符合 `2x4` 主区域 <= 4。
- 视觉焦点：左侧大温度文本作为唯一主视觉锚点，暖绿到橙色渐变表达家庭关怀和天气提醒。
- 信息节奏：先看城市和温度，再看感冒指数与提示，最后触达拨号按钮。
- 关键横向关系：天气状态驱动关怀判断，右侧拨号作为行动出口，不与标签挤在同一行。
- 必须完整显示：`深圳`、温度文本、`感冒指数`、指数值、`打电话`、手机号动作参数。
- 拥挤 Row 宽度预算：root 内宽 `296vp`，三列间距 `16vp`，左列 `104vp`，右列 `58vp`，中列约 `118vp`。中列 `indexRow` 内部 padding 后约 `98vp`，`感冒指数` 约 `48vp`，间距 `4vp`，`待评估/较易感冒` 约 `42-48vp`，可完整显示。右侧按钮宽 `54vp`，`打电话` 约 `36vp`，可完整显示。
- 交互和 DataModel：`Button.onClick` 使用 EventHandler 数组，参数从 `/contact` 读取，不在事件中硬编码业务参数。
- CardSpec：`suggestSize` 为 `2x4`；使用已声明天气能力 `weather.overview.get`，参数为深圳、1 天预报，结果写入 `/data/weather`；刷新计划安装、可见和 30 分钟周期刷新。

## 显式改进

- 内部首版把右侧动作列设为 `48vp` 且按钮标签较窄，存在 CTA 挤压风险；改为 `58vp` 动作列和 `54vp` 按钮，减少中间文案长度。
- 内部首版想把感冒指数作为 CardSpec capability 声明，但参考能力只声明了天气和日历；最终不虚构感冒指数能力，改为宿主数据字段假设。

## 宿主假设

- 点击拨号使用宿主 catalog 自定义函数 `makePhoneCall`，参数为 `phoneNumber` 和 `displayName`。该函数不是 A2UI 预定义函数，需要宿主实现并注册。
- 感冒指数展示字段 `/care/coldIndexText` 由 CreateMyCard 宿主技能或业务侧根据天气/健康规则更新；当前 CardSpec 不虚构未声明的感冒指数 capability。

## 校验日志

- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-xhigh\case-1\genui.jsonl`：通过。
- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-xhigh\case-1\cardspec.json`：通过。
- 最终验收：按 `review-validation.md` 与 `design-review.md` 检查，尺寸、主区域数量、受保护文本、点击事件、CardSpec 数据路径对齐均满足要求。
