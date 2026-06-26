# Case 2 — 天气关怀卡片 生成记录

## 用户需求

天气关怀卡片：深蓝→天空蓝渐变 + 毛玻璃质感；显示深圳当前温度 31° 与天气情况；
高温预警明显提醒；健康提示（感冒指数 易发）提醒家人；旁边一键拨打电话联系父母；
整体信息清晰、重点一眼可见。

## 场景与模式识别

- 场景分类：`status glance` + `action card` 复合，落在构图规则的「家庭关怀告警」场景。
- 模式：模式 1（一句话生成新卡片，无既有 DSL）。
- 形态：动态卡片（实时天气来自端侧能力），高温预警 / 健康提示 / 拨打电话为宿主静态配置。

## 尺寸选择

选择 **2x4 / 320 x 160vp**。理由：

- 需要承载多个并列要点：定位+温度+天气、高温预警、感冒指数、拨打电话按钮。
- 天然左右关系：左侧 = 天气主指标焦点面板；右侧 = 预警/健康/动作细节栈。
- 受保护文本（温度、预警标签、感冒指数、CTA「一键拨打电话」）在 2x2 的 160vp 宽里会互相挤压；2x4 给受保护文本更安全的横向空间。
- 主区域数 = 4（focal、alert、health、action），正好等于 2x4 上限 ≤4，不滚动。

## 语义角色

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | 📍深圳 | locationRow（Text + Text） |
| primaryAnswer | 31°（46fp 大字） | tempText |
| metric/context | 晴 高温 | conditionText |
| status | ⚠ 高温预警（琥珀色徽章） | alertBadge |
| context | 感冒指数 · 易发 | healthRow |
| action | 一键拨打电话 → makePhoneCall | callButton |

## 视觉构图理由

- Root 用 `Stack`：底层渐变 shell（深蓝 #0B1E4D → 中蓝 #1E50A8 → 天空蓝 #5BB6E8，RightBottom 方向）+ 一个浅蓝圆形光晕层 `glow`，再叠内容 Row。满足「深蓝到天空蓝渐变」需求。
- 毛玻璃质感：左侧 focalPane 与右侧各信息块使用半透明白底 `#24FFFFFF`/`#1FFFFFFF` + 1px 高光边框 `#3DFFFFFF`，在渐变上形成磨砂玻璃面板效果（Form 子集无原生 blur，用半透明块+高光边框+光晕模拟）。
- 视觉焦点唯一：左侧 46fp「31°」为主导锚点；高温预警徽章为次级视觉装置（暖色 `#33FF7A1A` 底 + `#80FFB060` 边框 + ⚠ 字形），与冷色蓝形成强对比，确保「明显提醒」。
- 信息节奏：左焦点面板（定位/温度/天气）↔ 右细节栈（预警 → 健康 → 动作），由上到下重要性递减。

## 关键横向关系与宽度预算

contentRow 内宽 = 320 − padding(12×2) = 296vp；itemMargin 10。focalPane 134vp + detailPane 150vp + 10 ≈ 294vp，放得下。

detailPane（150vp）内拥挤行预算：

- alertBadge：内宽 150 − padding(8×2)=134vp。⚠(~15) + margin4 + 「高温预警」(4×~14≈56) ≈ 75vp，单行完整。
- healthRow：内宽 134vp。「感冒指数」(layoutWeight:1，弹性) + margin6 + 「易发」(2×~14≈28)。标签得余量 ~100vp，4 字充裕。
- callButton：「一键拨打电话」6 字 ~90vp，在 150vp 面板内单行。

focalPane（134vp，padding12，内宽 ~110vp）：「31°」46fp ≈ 70vp 放得下并设 minFontSize 34 兜底；「晴 高温」15fp ≈ 60vp 单行。

所有受保护文本 `textOverflow:"none"`、`maxLines:1`，不依赖 ellipsis/clip。

## 交互与 DataModel 形状

- 唯一主动作：`callButton`，`onClick` 调用宿主函数 `makePhoneCall`，参数 `phoneNumber` / `contactName` 从 DataModel 的 `/action/call` 绑定，未硬编码业务号码（电话号为空字符串占位，由宿主/用户配置填入）。
- DataModel 按语义分组：
  - `/data/weather`（动态，端侧能力写入）：location、current。
  - `/data/alert`（静态宿主配置）：高温预警标签。
  - `/data/health`（静态）：感冒指数 tip。
  - `/action/call`（静态）：拨打电话动作参数。
  - `/state/loading`。

## CardSpec 契约

- `suggestSize: "2x4"`，与 DSL 尺寸一致。
- 动态能力 `weather.overview.get`（v1.0），`arguments: {districtName:"深圳", forecastDays:1}`，`writeResultTo:/data/weather`。
- UI 绑定 `/data/weather/location/districtName`、`/data/weather/current/temperatureText`、`/data/weather/current/condition` 均可由 outputSchema 推导。
- `bindingId: shenzhen_weather` + `refreshPlan`（onInstall / onVisible / periodic 1800s）让深圳天气定时刷新。
- 预警 / 健康 / 拨打电话为静态宿主数据，放在 `/data/weather` 之外的兄弟路径，避免被天气刷新覆盖。

## 显式改进（至少一次）

**第一版问题：** 初版把高温预警写在 `/data/weather/alert/label`，并由 weather 能力 `writeResultTo:/data/weather` 整体覆盖。但 `weather.overview.get` 的 outputSchema 没有 `alert` 字段——端侧刷新会把整个 `/data/weather` 用归一化结果替换，导致 `alert` 在运行时被清空，高温预警无法显示，违反 CardSpec 对齐阻塞项（UI 绑定路径必须可由 writeResultTo + outputSchema 推导）。

**改进：** 将高温预警迁移到独立的静态路径 `/data/alert/label`（与 `/data/weather` 平级的兄弟路径），天气能力不会覆盖它；同时保持温度/天气/定位由动态能力驱动。这样既满足「高温预警明显提醒」的用户需求，又保证数据契约自洽。其余还强化了预警徽章的暖色对比与边框，使重点更突出。

## 校验日志

```
$ python scripts/validate_genui_card.py .../case2/card.genui
GenUI 卡片校验通过
消息数：3
组件数：17
surfaceId：weather-care-card

$ python scripts/validate_cardspec.py .../case2/card.cardspec.json
CardSpec validation passed
```

改进修复后重新校验，两个脚本均无错误、无警告通过。

## 最终验收结论

- 协议合法性：genui 校验通过（v0.9、extended catalog、单次 updateComponents、root 可解析、仅 Form 10 组件、Text.content/Button.label/Image 规则、无网络/SVG/占位媒体、无 $__widthBreakpoint）。
- 卡片形态：明确 2x4；主区域 = 4（≤4）；无滚动/段落/表格；由语义角色与构图规则构造，非模板改造。
- 受保护文本：温度、预警标签、感冒指数/易发、CTA 全部单行完整显示，无 ellipsis/clip。
- 交互：callButton 有真实 onClick（makePhoneCall），参数由 DataModel 绑定。
- CardSpec：suggestSize 一致；动态 binding 来自已声明能力；writeResultTo 在 /data 下；UI 路径可推导；refreshPlan 仅引用已存在 bindingId。
- 宿主假设：`makePhoneCall` 视为宿主 catalog 已声明的自定义函数；电话号码留空由宿主/用户配置。

验收通过，可交付。
