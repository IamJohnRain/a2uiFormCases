# Layout Notes

- 场景/模式：status glance + action card，模式 1 新卡片。
- 尺寸：选择 `2x4`，因为天气主指标、明显高温预警、健康提示和拨号动作需要横向分区；`2x2` 会挤压告警和 CTA。
- 语义角色：identity=`深圳`，primaryAnswer=`31°`，context=`晴热 体感偏高`，status=`高温预警/注意防暑补水`，context=`感冒指数：易发`，action=`拨打`。
- 主区域：左侧天气主指标，中间关怀信息，右侧联系动作，共 3 个主区域，未超过 `2x4` 上限。
- 视觉焦点：深蓝到天空蓝渐变 shell + 42fp 温度；半透明告警块和健康块制造毛玻璃质感。
- 信息节奏：先看到 `31°`，再看到高温预警，再看到健康提示和拨号动作。
- 关键横向关系：天气状态与家庭关怀并列，拨号动作作为从属但清晰的右侧 CTA。
- 必须完整显示：`深圳`、`31°`、`晴热 体感偏高`、`高温预警`、`注意防暑补水`、`感冒指数：易发`、`拨打`。
- 宽度预算：root 320vp，内容层宽 296vp；主 Row 三列 `104 + 128 + 48 + 2*8 = 296vp`。天气列中 `31°` 104vp 可容纳 42fp 短温度；中间块宽 128vp，内部 padding 16vp 后可用 112vp，足够 6-7 个中文字短语；右侧按钮宽 48vp，`拨打` 可完整显示。
- 交互/DataModel：`Button.onClick` 调用 `hostMakePhoneCall`，参数从 `/family/phoneNumber` 和 `/family/contactName` 绑定；DataModel 按 `weather`、`alert`、`health`、`family`、`action` 分组。
- CardSpec：`suggestSize=2x4`。本卡使用静态展示数据，不声明 `dataBindings`，避免虚构未声明的高温预警和感冒指数能力。
- 显式改进：首版计划把健康提示和拨号按钮放在同一中间行，容易让 `感冒指数：易发` 与 CTA 互相挤压；最终改成中间双玻璃块 + 右侧独立按钮，保证受保护短语完整显示。
- 宿主假设：`hostMakePhoneCall` 是宿主 catalog 明确注册的自定义函数，用于发起拨号或跳转拨号确认页；Form DSL 不使用预定义函数。

# Validation Log

- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_genui_card.py gpt5.5-xhigh\case-2\genui.jsonl`：通过。消息数 3，组件数 19，surfaceId `weather-care-card`。
- `python .opencode\skills\harmony-card-generation-v5\scripts\validate_cardspec.py gpt5.5-xhigh\case-2\cardspec.json`：通过。
