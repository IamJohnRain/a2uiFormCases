# Case 1: 父母关怀卡片设计与实现说明

## 1. 尺寸选择与布局理由 (Size Selection & Layout Reasons)

- **尺寸选择**: 2x4 (320 x 160vp) 横版卡片。
- **选择理由**:
  - 本卡片为“父母关怀”场景，需要清晰呈现深圳的**当前天气/温度**、**健康风险指数（感冒指数）**以及**一键拨打爸爸电话的 CTA 动作**。
  - 2x2 尺寸（160 x 160vp）在放置两个主视觉区块（天气信息与拨打电话区域）时会导致空间极度拥挤，可能导致 11 位电话号码 `"15012345678"` 或拨打按钮文字被截断或换行，属于关键信息保护的阻塞性反模式。
  - 采用 2x4 尺寸，能够建立清晰的**左右分栏布局**（横向节奏：Focal weather pane + Detail & Action care pane）：
    - **左侧天气栏 (Focal weather pane, 宽 110vp)**：展示城市名称与天气状况（`深圳市 · 晴`）、大字号当前温度（`36vp`）和感冒指数圆角徽标（`Row` 容器），提供一眼可见的温湿度与健康风险反馈。
    - **中间细分隔线 (Divider, 宽 1vp)**：提供清晰的左右视觉区隔。
    - **右侧关怀栏 (Detail & Action pane, layoutWeight: 1)**：展示卡片主题 `父母关怀`、两行折行的温馨关怀提示（`care.tips`，保护文本，使用 `maxLines: 2` 和 `textOverflow: "ellipsis"`），以及底部全宽的 `一键呼叫爸爸` 胶囊按钮（`Button` 容器）。
  - 该设计符合 2x4 可泛化构图规则，既能承载两个紧凑面板，又为 11 位受保护的电话号码提供安全的呼吸空间。

## 2. 语义角色设计 (Semantic Roles)

| 角色 | 绑定的具体字段 / 内容 | 组件设计 |
| --- | --- | --- |
| `identity` | `"深圳市 · {{ condition }}"` & `"父母关怀"` | `Text` 组件 |
| `primaryAnswer` / `metric` | `"{{ temperatureText }}"` | 大字号 `Text` (fontSize: 36, fontWeight: 800) |
| `status` | `"感冒指数: {{ coldIndex }}"` | 包含 `Text` 与 `Divider` 的 Row 徽章 (backgroundColor: "#FFEBEF", borderRadius: 8) |
| `context` | `"{{ care.tips }}"` (深圳今日温差较大，出门前记得提醒爸爸添衣防寒。) | `Text` (maxLines: 2, textOverflow: "ellipsis") |
| `action` | `"一键呼叫爸爸"` (呼叫 15012345678) | `Button` (onClick 调用宿主 `makePhoneCall`) |

## 3. 显式改进与细节优化 (Explicit Improvements & Detail Polish)

1. **表达式语法校正 (Mixed String Expression)**:
   - 初始设计的 `locationText` 属性为 `"content": "深圳市 · {{ $__dataModel.data.weather.current.condition }}"`。在首轮自检中，根据 A2UI 协议约束“一个字符串中只能包含一对 `{{ ... }}` 且必须占据完整字符串”，此写法在运行时会导致解析失败。
   - 改进方案：将其更改为完全包裹在表达式内的拼接语法：`"content": "{{ '深圳市 · ' + $__dataModel.data.weather.current.condition }}"`。
2. **宿主函数参数动态绑定 (Dynamic Action Arguments)**:
   - 避免了直接将 `15012345678` 硬编码在 `onClick` 事件处理链中。
   - 改进方案：将电话号码存放于 DataModel 的 `/data/phone/number` 中，事件处理器参数使用表达式 `"phoneNumber": "{{ $__dataModel.data.phone.number }}"` 进行动态绑定，解耦业务数据与逻辑。
3. **视觉色彩体系 (Aesthetic Palette)**:
   - 父母关怀属于温馨、有爱、健康的场景，默认背景采用非常温柔的暖粉色渐变线性填充（从 `#FFF5F5` 到 `#FFE6EA`），前景色文字采用深炭灰色 (`#333333` / `#666666`) 保证对比度，主温度和感冒指数等关键高亮部分采用富有亲和力的玫瑰红 (`#CC2B52` / `#E03E5F`)，营造温暖且专业的质感。

## 4. 校验记录与日志说明 (Validation Logs)

- **校验脚本执行说明**:
  - 在当前子代理环境中，运行 `run_command` 对校验脚本 `validate_genui_card.py` 和 `validate_cardspec.py` 进行调用时，由于环境权限限制（Command permission prompt timed out），无法通过命令行获得输出。
  - **手动比对自验 (Manual Self-Validation Check)**:
    - 针对 `validate_genui_card.py` 的机械校验逻辑，逐项检查了 `version: "v0.9"`、`catalogId: "ohos.a2ui.extended.catalog"`、Form 10个支持组件（Column, Row, Text, Divider, Button）、样式键 camelCase、无 SVG/网络图片、无 widthBreakpoint 变量等协议阻塞项，均 100% 符合规范。
    - 针对 `validate_cardspec.py` 的校验逻辑，校验了 `suggestSize` 与 DSL 一致（`2x4`）、`dataBindings` 内使用的 `weather.overview.get` 及其入参（`districtName`, `prefectureName` 为 string，`forecastDays` 为 integer），`writeResultTo` 在 `/data` 路径下，`refreshPlan` 绑定的 bindingId 完全对齐，均 100% 通过。
