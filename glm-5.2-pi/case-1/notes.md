# Case-1 · 父母关怀卡片（深圳温度 + 感冒指数 + 一键呼叫爸爸）

> 用户一句话：帮我创建一个父母关怀卡片，显示深圳的温度与感冒指数，并支持一键给我爸（15012345678）打电话。
> 注：用户提到的 “CreateMyCard 技能” = 本卡片生成技能（harmony-card-generation-v5），按其工作流执行。

## 1. 场景与模式

- 场景分类：`家庭关怀告警`（status glance + action card）—— 天气状态 + 关怀风险 + 联系动作。
- 模式：`模式 1：一句话桌面卡片`（新建卡片，用户未提供已有 DSL）。

## 2. 尺寸选择：`2x4`（320 × 160vp 横版）

依据 `card-composition-rules.md` 场景映射：家庭关怀告警在 “建议/联系/详情重要” 时使用 `2x4`，结构为 “状态/天气 + 风险详情 + 联系动作”。横向天然存在左右关系（左：天气速览；右：感冒指数关怀 + 呼叫动作），且受保护内容（温度、感冒级别、CTA）需要比 2x2 更安全的横向空间，避免呼叫按钮与感冒级别挤在同一狭窄列。

## 3. 语义角色与主区域

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | 深圳 | `locLabel` |
| primaryAnswer | 当前温度（大字） | `tempValue`（视觉焦点） |
| context | 天气现象（多云/小雨…） | `condLabel` |
| status | 感冒指数级别（易发） | `coldBadge` |
| context | 关怀建议 | `coldAdvice` |
| action | 一键呼叫爸爸 | `callBtn`（Button + onClick） |

主区域 3 个（天气焦点 / 感冒指数关怀 / 呼叫动作），≤ 2x4 上限 4。唯一视觉焦点 = 左侧大温度。

## 4. 布局理由（构图 + 宽度预算）

- Root：`Row`，width 320 / height 160，padding 12，圆角 22，`clip`。背景暖色 `linearGradient`（珊瑚 #FF8A5C → 粉 #FF6E8A → 紫 #7A5BE0），贴合 “关怀/温情” 场景；不继承任何示例配色。
- Root 横向预算：内宽 320 − 24(padding) = 296；两 pane `layoutWeight:1` + `itemMargin:18` → 每个 ≈ 139vp。
- 左 pane（weatherPane，Column，居中）：`locLabel` → `tempValue`(38fp) → `condLabel`，itemMargin 6。温度是唯一大视觉锚点。
- 右 pane（carePane，Column，spaceBetween）：`coldHeader`（感冒指数 + 易发徽标）顶 → `coldAdvice`（≤2 行）中 → `callBtn` 底。spaceBetween 让 CTA 独占底部行，不与徽标争抢。
- 拥挤 Row 宽度预算：
  - `coldHeader`（宽 ≈139vp）：`感冒指数`(4字@12fp≈48vp) + itemMargin 6 + `易发`徽标(~32vp) ≈ 86vp，spaceBetween 留 ~53vp 间隙，无挤压。
  - `callBtn`（宽 ≈139vp）：`呼叫爸爸`(4字@14fp≈56vp)，充分容纳，无截断。
- 受保护文本均无 `ellipsis/clip/marquee`：温度用 `minFontSize:28` 兜底，CTA/级别/现象按完整宽度计划显示。

## 5. 数据语义（重要：感冒指数的降级处理）

- **温度 + 天气现象**：真实动态数据，来自已声明能力 `weather.overview.get@1.0`，参数 `prefectureName:"深圳"`、`forecastDays:1`，`writeResultTo:"/data/weather"`。UI 绑定 `/data/weather/current/temperatureText`、`/data/weather/current/condition`、`/data/weather/location/prefectureName`，均可由 `writeResultTo + outputSchema` 推导。
- **感冒指数**：已声明能力 `weather.overview.get` 的 `outputSchema` **不含感冒指数字段**。按协议不捏造能力，故降级为**静态关怀文案**（`/data/care/coldLevel:"易发"`、`/data/care/coldAdvice`），初始值由 `updateDataModel` 提供。真实感冒指数需端侧补充 capability manifest（例如 `weather.index.cold`）或客户端按温度/湿度推导；当前为先满足用户可见需求的受限降级。
- **联系电话**：用户显式提供（15012345678），作为用户输入初始化在 `/data/contact.phone`，事件参数从 DataModel 绑定，未在 EventHandler 中硬编码。

## 6. 显式改进（输出前至少一次）

第一版（设想）：尝试 `2x2`，把温度/感冒级别/呼叫按钮竖向堆叠 → 温度被迫缩小，呼叫按钮与感冒级别同行挤压，CTA 标签可能被压缩。

改进：
1. 升级为 `2x4` 横版，建立 “天气 ⟷ 关怀+呼叫” 左右关系，温度保持大字焦点。
2. 右 pane 用 `spaceBetween` 把感冒头/建议/呼叫按钮分到上/中/下，CTA 独占一行，不再与徽标争抢。
3. 电话号从 DataModel 绑定（非事件硬编码）；温度加 `minFontSize` 兜底；location 预置 “深圳” 保证加载期身份稳定。
4. 对感冒指数采用诚实降级（静态关怀文案 + 文档说明），而非伪造动态能力。

## 7. 交互与宿主假设

- `callBtn`：`Button.label="呼叫爸爸"`，`onClick` 真实 EventHandler `[{call:"makePhoneCall", args:{phoneNumber: 数据绑定}}]`。
- `makePhoneCall` 为**宿主 catalog 自定义函数假设**（visual-interaction.md 列出的有意义宿主函数示例）。`call` 非表达式，`args.phoneNumber` 用表达式绑定。

## 8. CardSpec 契约

- `suggestSize:"2x4"`，与 DSL 一致。
- 动态：`dataBindings` 声明 `sz_weather`（weather.overview.get）。
- `refreshPlan`：`onVisible` + 周期 1800s，贴合关怀天气卡的刷新需求；仅引用已存在 `bindingId`。
- 静态关怀/联系数据不含 `dataBindings`，由初始 DataModel 承载。

## 9. 校验日志

```
=== GenUI validation (scripts/validate_genui_card.py) ===
GenUI 卡片校验通过
消息数：3
组件数：11
surfaceId：parents_care_weather
（无 error，无 warning）

=== CardSpec validation (scripts/validate_cardspec.py) ===
CardSpec validation passed
（无 error，无 warning）
```

## 10. 最终验收（review-validation.md）

- 协议阻塞项：version v0.9 ✓；catalogId `ohos.a2ui.extended.catalog` ✓；无 theme ✓；单次 updateComponents ✓；surfaceId 一致 ✓；id 唯一且引用可解析 ✓；仅用 Form 10 组件 ✓；用 `Text.content/Button.label` 非 `text/child/action` ✓；仅 onClick ✓；无网络/SVG/占位 URL ✓。
- 卡片阻塞项：明确选 2x4 ✓；主区域 3 ≤ 4 ✓；无滚动/长列表/页面 ✓；规则驱动非模板 ✓；可点击区有真实 onClick ✓；有布局理由 + 显式改进 ✓。
- CardSpec 阻塞项：cardspec 为 object ✓；suggestSize 与 DSL 一致 ✓；动态卡含 dataBindings ✓；capabilityId/arguments 来自已声明能力 ✓；writeResultTo 在 /data 下 ✓；UI 路径可由 writeResultTo+outputSchema 或初始 DataModel 推导 ✓；refreshPlan 仅引用已存在 bindingId ✓。
- 受保护文本换行：温度/感冒级别/CTA/现象均完整显示，无 ellipsis/clip/marquee ✓。

结论：校验与最终验收均通过，交付文件 `card.genui` + `card.cardspec.json`。
