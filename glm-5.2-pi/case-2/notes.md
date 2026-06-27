# Case-2 天气关怀卡片 — notes

## 用户需求
- 整体深蓝→天空蓝渐变 + 毛玻璃质感
- 深圳当前温度 31° + 天气情况
- 有高温预警要明显提醒
- 健康提示（感冒指数"易发"），提醒家人注意身体
- 旁边放一键拨打电话按钮，随时联系父母
- 信息清晰，一眼看到重要内容

## 场景与模式
- 场景：`家庭关怀告警`（status glance 天气 + action 呼叫 + information summary 健康提示）
- 模式：模式 1（一句话桌面卡片，无已有 DSL）

## 尺寸选择：`2x4` / 320×160vp
依据 card-composition-rules 场景映射"家庭关怀告警：状态/天气 + 风险详情 + 联系动作 → 2x4"。
- 左右关系：左=天气状态，右=健康详情+联系动作。
- 受保护关键信息（31°C、预警标签、感冒易发、CTA 标签）需要比 2x2 更安全的横向空间。
- 主区域 = 4（预警条 / 天气焦点 / 健康提示 / 呼叫按钮），等于 2x4 上限，未超标。

## 语义角色 → 组件
| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | 深圳 | locText |
| primaryAnswer | 31°C（42fp，主导焦点） | tempText |
| status/alert | ⚠ 高温橙色预警（醒目琥珀色） | warnBanner（整宽顶部条） |
| context | 晴 · 湿度 70% | condText + humidText |
| context/advisory | 感冒指数 易发（毛玻璃面板） | healthBlock |
| action | 拨打父母电话（毛玻璃胶囊 CTA） | callButton |

## 布局理由
- Root 为 `Stack`：直接承载 `linearGradient`（深蓝 #102A6E → 中蓝 #2A63BF → 天空蓝 #56A8E8，方向 RightBottom），单一子层 `overlay` 承载内容。
- overlay 为 `Column`（padding 11）：顶部整宽预警条 + 下方内容行（layoutWeight 填满剩余高度）。
- mainRow 为 `Row`：leftPane（layoutWeight 1 ≈ 172w）放天气焦点，rightPane（固定 116w）放健康提示 + 呼叫按钮。
- leftPane `justifyContent:center` 让温度数值成为视觉中心；rightPane `spaceBetween` 让健康面板在上、呼叫按钮在下。

### 视觉焦点与层级
- 31°C 是最大元素（42fp 白字），主导焦点。
- 预警条为饱和琥珀色（#F59E0B）整宽置顶 = 最先入眼，满足"明显提醒"。
- 健康提示用半透明白面板（#33FFFFFF + #66FFFFFF 描边）= 安静的提示，不与预警争抢。
- 呼叫按钮用近不透明白胶囊（#F2FFFFFF）+ 深蓝字 = 清晰可点的 CTA。

## 显式改进（必做的一次迭代）
- v1（内部）：两栏横排，预警做成左栏底部的小徽章。
  问题：① 预警不够醒目，与用户"明显提醒"诉求不符；② 左栏塞 地点+温度+天气+预警 四项，136vp 内偏挤。
- v2（改进，即当前版）：把预警升级为卡片顶部整宽条（最高视觉优先位置、整宽、饱和琥珀色、⚠ 字形），使其无法被忽略；下方改为两栏内容行（左天气焦点、右健康+呼叫）。这样层级更清晰（条=告警 / 左=温度焦点 / 右=健康+动作），更好满足"明显提醒"与"一眼看到重要内容"，且通过把预警条压到 ~26vp、内容行用 layoutWeight 填充，把四区域稳稳压进 138vp 可用高度。

## 受保护内容宽度预算（overlay 内宽 ≈ 298vp；左栏 ≈172w，右栏 ≈116w）
- warnBanner：`⚠`(~16) + 间距 5 + `高温橙色预警`(12fp×6≈72) + padding 10 ≈ 103px < 298 ✓
- tempText：`31°C`(42fp×4≈96) < 172 ✓
- condRow：`晴`(13) + 6 + `湿度 70%`(`'湿度 '+70+'%'`≈80) ≈ 99 < 172 ✓
- healthBlock：`感冒指数`(11fp×4≈44)、`易发`(20fp×2≈40) < 内宽 96 ✓
- callButton：`拨打父母电话`(14fp×6≈84) < 内宽 ≈100 ✓
- 全部受保护文本均 `maxLines:1`，未使用 ellipsis/clip/marquee。

## 拥挤行预算
- mainRow：298 = leftPane(layoutWeight,≈172) + itemMargin 10 + rightPane(116) ✓
- 右栏垂直：healthBlock(≈54) + callButton(≈38) ≈ 92 < mainRow 高度 ≈108 ✓
- 左栏垂直：loc(15)+3+temp(42)+3+cond(15)=78，居中于 ≈108 ✓

## 交互 / DataModel
- callButton.onClick → `makePhoneCall`（宿主假设函数），`phoneNumber` 绑定 `action.parentPhone`，业务号码不硬编码于事件。
- DataModel 按领域分组：`data.weather.*`、`health`、`alert`、`action`。

## CardSpec 契约
- `suggestSize: "2x4"`，与 DSL 一致。
- 动态部分：`weather.overview.get` v1.0，参数 `prefectureName:"深圳"`、`forecastDays:1`，`writeResultTo:/data/weather`，bindingId `shenzhen_weather`。
- 刷新：onInstall / onVisible / 周期 1800s，均引用已存在 bindingId。
- UI 路径对齐：`data.weather.current.temperatureText/condition/humidityPercent`、`data.weather.location.prefectureName` 均可由 `/data/weather + outputSchema` 推导。

## 校验日志
- `validate_genui_card.py card.dsl.jsonl` → **GenUI 卡片校验通过**（消息数 3，组件数 17，surfaceId weather-care-card，0 错误 0 警告）。
- `validate_cardspec.py cardspec.json` → **CardSpec validation passed**（0 错误 0 警告）。
- 设计评审 + 受保护文本评审：全部通过，未触发修改，无需重新校验。

## 能力边界与受限例外（诚实声明）
1. **高温预警、感冒指数无已声明能力**：`weather.overview.get` 的 outputSchema 只含 current/daily/location/updatedAt，没有 alerts/warning 字段，也没有健康指数能力。因此 `alert.*` 与 `health.*` 为**静态演示值**（满足用户"看到 31°/易发/预警"的演示需求），卡片同时用真实 `weather.overview.get` 能力驱动天气数值。要使预警/健康指数真正动态，需端侧补充 weather-alert / health-index capability manifest。
2. **31°C 本身不构成高温预警**（国标高温预警阈值 ≥35°C）；此处预警条为"明显提醒"的视觉演示。生产中应由真实预警能力驱动，或在温度达阈值时由端侧逻辑触发。
3. **毛玻璃质感为模拟**：Form 样式子集不含 `backdropBlur`，故毛玻璃用半透明白块（#33FFFFFF 背景 + #66FFFFFF 描边 + 柔和阴影）+ 渐变底模拟；预警条为优先醒目而用实色琥珀。
4. **makePhoneCall 为宿主假设函数**：需宿主 catalog 注册；`action.parentPhone` 为占位演示号码（13800000000），生产应由通讯录/应用数据提供真实号码。
5. **无真实本地图片资源**：未使用 Image，全部视觉来自渐变 + 半透明块 + 文本，符合"无素材时使用非媒体视觉技法"。
6. 未加 accessibility.label：Form 子集未文档化该属性，避免引入未声明属性；如宿主支持可后续补充。
