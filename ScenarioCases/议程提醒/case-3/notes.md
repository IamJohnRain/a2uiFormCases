# case-3 会议入会卡片 — 设计与校验记录

## 需求
显示：会议时间、会议 ID、实时屏幕共享内容、聊天消息。

## 取舍
- 主答案：当前会议（产品周会，进行中）。
- 4 个信息职责互斥：时间点/段（time）、标识（id）、共享内容（share）、聊天消息（chat），各由一个组件主承载，无单位/别名重复。
- 4 个并列信息组无法在 2x2 安全区放下 → 升级 **2x4**（hero-left split）。

## 尺寸与布局预算（2x4，内容区 276×116，root padding 12）
- 横向：左列 150 + itemMargin 10 + 右面板 116 = 276 ✓
- 左列 Column（150×116，居中）：4 行，itemMargin 8。
  - title_row 22 + time_row 18 + id_row 18 + status_row 18 + 3×8 间距 = 100 ≤ 116 ✓
  - 每行内：图标(16~18) + gap6 + 文本宽度 = 150 ✓（文本 width 126/128/136 已扣图标与 gap）
- 右面板 share_panel（116×116，padding 10，内容宽 96）：
  - header 14 + content 28(2行×14) + chat 28(2行×14) + 2×8 间距 = 86 ≤ 96 内容高 ✓
- 间距尺度只用 6/8/10/12；组内(6/8) < 组间/面板 padding(10/12) ✓

## 受保护文本
- time（14fp）、id（12fp）、status（12fp）给足固定宽度、none 截断策略，完整显示。
- title 为可选可截断（ellipsis）；share/chat 为实时长文本，允许 2 行 + ellipsis（非受保护）。

## 颜色来源
- root 背景 = background_primary(light) → #FFFFFFFF
- 标题/时间 = font_primary → #E5000000；ID = font_secondary → #99000000；chat = font_tertiary → #66000000
- 共享/进行中强调（live_dot、status_text）= brand → #FF0A59F7（会议进行真实状态，单一主色族）
- 右侧共享面板 = comp_background_secondary(light) → #19000000（中性背板承载分组）
- 同卡仅一个主场景色族（brand 蓝），无第二高饱和色。

## 能力说明（重要）
- 事件能力 manifest 中无"入会/会议 app"deeplink 目标，data-capability 目录无会议数据能力。
- 因此**不伪造** join 的 onClick `call`，也**不虚构** dataBindings/refreshPlan。
- 卡片为静态降级：实时字段（share.state/content、chat.*）由端侧后续 updateDataModel 刷新即可，UI 路径均可从初始化 DataModel 推导。
- 如需"点击入会"或真实实时拉取，需端侧补充对应 event-capability 与 data-capability manifest。

## 协议自检
- genui 三行：createSurface / updateComponents / updateDataModel ✓
- version v0.9，catalogId extended，2x4：root width300/height140/borderRadius22/clip true，CardSpec suggestSize 2x4 一致 ✓
- 仅用 Form 组件（Row/Column/Text/Image/Stack）；图片均为素材库声明路径 + contain ✓
- 所有表达式引用字段在 updateDataModel.value 中存在 ✓
