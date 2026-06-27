# case-1 天气关怀卡片

## 布局理由

- 场景/模式：新建卡片，属于 `status glance + action card`；采用模式 1。
- 尺寸：选择 `2x4`，因为温度、天气、感冒指数和通话 CTA 都是关键信息，`2x2` 会挤压受保护文本。
- 语义角色：identity 为“天气关怀”；primary answer 为当前温度；metric 为温度和湿度；context 为天气、感冒指数；action 为“立即通话”。
- 主区域：左侧温度焦点、中间天气详情、中间底部 CTA，共 3 个主区域，符合 `2x4 <= 4`。
- 视觉焦点：左侧半透明焦点块内的大温度，天气字形只作轻量场景锚点。
- 信息节奏：先读温度，再读天气和湿度，再读感冒指数，最后执行通话。
- 关键横向关系：`metric-context-action`，温度和关怀建议并列，通话动作从属于关怀上下文。
- 必须完整显示的信息：温度、天气、感冒指数、“立即通话”、标题和状态标签均未使用 ellipsis/clip/marquee。
- 拥挤 Row 宽度预算：
  - root 内容宽度约 296vp；左 pane 108vp，间距 10vp，右 pane 178vp。
  - headerRow 可用 178vp，标题约 72vp，状态约 28vp，间距 6vp，余量约 72vp。
  - conditionRow 可用 178 - 16 padding - 10 gaps = 152vp；“天气”约 24vp，“加载中/小雨/多云”等约 52vp，湿度文本约 48vp，余量约 28vp。
  - coldRow 可用 178 - 6 gap = 172vp；“感冒指数”约 56vp，“待评估/偏高/较低”等约 54vp，余量约 62vp。
  - callButton 宽 178vp，“立即通话”约 56vp，完整显示安全。
- 交互和 DataModel：`Button.onClick` 调用宿主函数 `makePhoneCall`，号码和原因从 `/action` 读取；天气字段从 `/data/weather` 读取；感冒指数当前作为关怀域 `/care/coldIndexText`。
- CardSpec：`suggestSize` 为 `2x4`；动态能力使用已声明的 `weather.overview.get`，参数 `forecastDays: 1`，写入 `/data/weather`；刷新计划为安装、可见和 30 分钟周期刷新。

## 一次显式改进

第一版把通话 CTA 放在右侧窄栏，虽然可读但会让“立即通话”与天气状态争宽。改进版改为两栏结构，并把 CTA 放在右侧详情底部的全宽按钮中，提升可点击面积和受保护文本安全性。

## 校验结果

- `python .agents/skills/harmony-card-generation-v5/scripts/validate_genui_card.py gpt5.5-codex-yyxcase/case-1/card.genui.jsonl`
- `python .agents/skills/harmony-card-generation-v5/scripts/validate_cardspec.py gpt5.5-codex-yyxcase/case-1/card.cardspec.json`
- 校验状态：通过。GenUI 校验输出 `GenUI 卡片校验通过`，CardSpec 校验输出 `CardSpec validation passed`。

## 宿主函数假设

- 假设宿主 catalog 已声明自定义函数 `makePhoneCall`，参数为 `phoneNumber` 和 `reason`，用于发起电话能力。
- 假设宿主在运行时填充 `/action/phoneNumber`；当前初始值为空字符串，避免在产物中写死真实号码。
- 已声明天气能力仅返回天气概要，不包含独立“感冒指数”能力；本卡片把 `/care/coldIndexText` 作为宿主业务归一化或静态关怀字段，后续可由宿主根据天气、湿度和本地健康策略更新。
