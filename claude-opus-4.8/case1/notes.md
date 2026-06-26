# Case1 父母关怀卡片 — 生成说明

## 用户需求
创建一个父母关怀卡片，显示深圳的温度与感冒指数，并支持一键给爸爸（15012345678）打电话。

## 场景 / 模式 / 尺寸
- 场景：家庭关怀 / 天气告警（status glance + action card 混合）。
- 模式：模式 1（一句话桌面卡片，新建，无现有 DSL）。
- 尺寸：**2x4 / 320x160vp**。card-composition-rules 明确建议家庭关怀类用 2x4；存在天然左右关系（左温度指标 / 右感冒指数 + 拨打动作），且需为受保护的电话 CTA 留出安全横向宽度。

## 能力范围确认
- 温度：可用已声明能力 `weather.overview.get`，属动态绑定（深圳，forecastDays=1）。
- 感冒指数：天气能力 outputSchema 仅含 temperature/condition/humidity/daily，**没有感冒指数字段**，故不虚构能力，将其作为本地配置 `/config/fluIndex` 展示。
- 打电话：宿主自定义函数 `makePhoneCall`（声明为**宿主假设**），电话号码绑定 DataModel `/config/phoneNumber`，不在事件处理器中硬编码。

## 语义角色
- identity：标题"关心爸爸" + 城市"深圳"
- primaryAnswer/metric：温度大数字（动态 `/data/weather/current/temperatureText`）
- context：天气现象 condition + 感冒指数 status 标签
- action：全宽"打给爸爸"Button -> makePhoneCall

## 布局理由
- root：Row(320x160)，padding 14，borderRadius 24，clip，深蓝背景。3 个直接子节点：metricPane | midDivider | rightPane。
- 节奏：metric-context-action 变体。左 metricPane 固定 118vp（城市/温度/天气一列），中间竖向 Divider(1vp)，右 rightPane layoutWeight:1 取剩余约 160vp。
- 视觉焦点：左侧温度 40fp 大数字（minFontSize 28 防变长溢出）。
- 右 pane：感冒指数橙色半透明标签块（标签 + 等级两行）+ 全宽绿色拨打 Button。
- Row 宽度预算：available = 320 - 28(padding) - 12(itemMargin) - 1(divider) ≈ 279vp；metricPane 118 固定，rightPane ≈ 150vp。CTA "打给爸爸" 4 字约 48-56vp << 150vp，安全完整显示。
- 受保护内容（温度文本、城市、感冒指数等级、CTA 标签）均 maxLines:1，无 ellipsis/clip。

## 显式改进
第一版内部草稿把温度与感冒指数都堆在左 pane、右 pane 只放一个按钮，导致左挤右空、层级失衡。改进：左 pane 仅承载温度焦点（温度+城市+天气现象），感冒指数移到右 pane 作为带背景的 status 标签条，下接全宽拨打按钮，使左右信息量均衡、焦点清晰、CTA 获得安全横向宽度。

## 校验日志
- `python scripts/validate_genui_card.py card.genui` -> GenUI 卡片校验通过（消息数 3，组件数 14，surfaceId care-dad-card），无警告。
- `python scripts/validate_cardspec.py card.cardspec.json` -> CardSpec validation passed，无警告。

## 最终验收结论
- 协议合法性：通过（脚本机械校验，0 错误 0 警告）。
- 卡片形态：2x4，主区域 3 个（≤4），无滚动/段落/页面化。
- CardSpec：suggestSize=2x4 与 DSL 一致；动态卡片含 dataBindings；capabilityId/arguments 来自已声明天气能力；writeResultTo=/data/weather 在 /data 下；温度 UI 路径可由 writeResultTo + outputSchema 推导。
- 受保护文本：温度、城市、感冒指数、CTA 标签均完整单行显示。
- 交互：拨打 Button 有真实 onClick EventHandler，号码绑定 DataModel。
- 宿主假设：`makePhoneCall` 为宿主自定义函数假设。
- 结论：通过，可交付。
