# Case5 睡眠监测桌面卡片 — 生成记录

## 用户需求

深紫色夜晚渐变风格的睡眠监测桌面卡片：顶部"睡眠不足"提示，中间突出睡眠时长"6小时45分"并用圆环表现睡眠状态，底部"设定早睡提醒"按钮（带闹钟图标），整体安静、柔和、有科技感与高级感。

## 场景与模式识别

- 场景分类：`status glance`（健康/睡眠进度速览）。对应 capability.md「睡眠/健康进度：状态、环形进度、时长、提醒动作」，在能力范围内。
- 模式：模式 1（一句话生成新卡片，用户未提供已有 DSL）。
- 交付物：一个 `genui` JSONL + 一个 `cardspec` JSON。

## 尺寸选择

- 选择 **2x2 / 160x160vp**。
- 理由：只有一个主答案（睡眠时长 + 圆环）、一个状态标签、一个动作，竖向节奏 `status -> 焦点(环+时长) -> action`。不存在需要 320vp 横向宽度的左右关系；圆环居中包裹时长，构成单一主导焦点。属于 `visual-primary-context` / `single-focal` 混合节奏。
- CardSpec `suggestSize` = `2x2`，与 DSL 一致。

## 语义角色

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| status | "睡眠不足" 警示徽章 | 带半透明紫色底的小 Text |
| primaryAnswer / metric | "6小时45分"（叠加在圆环中心） | Progress(ring) + Stack 居中 Text |
| context | "目标 8 小时" 极小 caption | 小 Text |
| action | "设定早睡提醒" 带闹钟字形的全宽 pill | 可点击 Row（onClick） |

## 布局理由（写 JSON 前）

- 视觉焦点：唯一主导锚点是 96x96 的 `Progress` 环，时长文本通过 `Stack`（alignContent center）叠加到环中心，使环真正"框住"主指标，而不是沦为附带小元素。
- 信息节奏：root Column `justifyContent: spaceBetween` + `itemMargin 10`，三区从上到下均衡分布，padding 12，可用内部约 136x136vp。
- 关键横向关系：底部 action 是一个 Row（闹钟字形 + 标签），无其他拥挤横向行。
- 必须完整显示的关键信息：状态"睡眠不足"、主时长"6小时45分"、CTA"设定早睡提醒" 全部 `maxLines:1` 且不使用 ellipsis/clip。时长另加 `minFontSize:16` 兜底。
- action Row 宽度预算：父内宽约 136vp，pill padding 9 两侧 → 内容约 118vp；闹钟字形约 15vp + itemMargin 6 + 标签"设定早睡提醒"(6 字 @13fp ≈ 80vp) ≈ 101vp < 118vp，单行可放下。
- 交互与 DataModel：action Row `onClick` 调用宿主函数 `openSleepReminderSetting`，`reminderId` 从 `$__dataModel.action.reminderId` 绑定，不硬编码业务 ID。DataModel 按语义分组 `sleep` / `action`，展示字符串预计算（durationText / targetText）。
- 视觉语言：深紫夜晚线性渐变 `#1B1733 -> #2A2150 -> #3A2E78`（RightBottom），环色 `#9E83FF`，文字为白/浅紫，对比充足；仅 1 个主要半透明块（action pill，符合 2x2 上限）。无真实本地图片素材，因此省略 Image，改用渐变 + Progress 环 + 半透明块表达，符合"无素材用非媒体技法"。

## 显式改进（至少一次）

- 第一版内部草稿把状态、环、时长、按钮做成四个竖向堆叠项，环较小且时长作为独立一行放在环下方 —— 这会埋没主答案，并让 Progress 环退化为一个很小的附带元素，违反表现力工具箱「Progress 应是主要焦点之一」。
- 改进：用 `Stack` 把时长文本叠加到环的正中心，让环真正包裹主指标，形成单一主导焦点；把环放大到 96x96；动作仅保留一个半透明 pill（2x2 上限），状态做成小号着色徽章而非第二个竞争块，从而保持唯一清晰层级。

## 校验日志

GenUI 校验（scripts/validate_genui_card.py，针对 card.genui）：

```
GenUI 卡片校验通过
消息数：3
组件数：10
surfaceId：sleep-monitor-card
```

CardSpec 校验（scripts/validate_cardspec.py，针对 card.cardspec.json）：

```
CardSpec validation passed
```

两个脚本均 0 错误、0 警告。

## 最终验收结论（review-validation.md）

- 协议阻塞项：JSONL 每行单 object；version 全为 v0.9；catalogId 为 ohos.a2ui.extended.catalog；createSurface 无 theme；仅一次 updateComponents；surfaceId 一致；无重复 id、引用全部可解析；仅用 Form 10 组件子集；使用 Text.content / Button 用 label（本卡未用 Button，用可点击 Row）；仅 onClick 事件；无 $__widthBreakpoint / 网络图 / SVG / 占位 URL。全部通过。
- 卡片阻塞项：明确选 2x2；主区域 3 个（≤3）；无需滚动/列表/段落；规则构造非模板改造；有布局理由 + 一次显式改进；可点击 Row 有 onClick。全部通过。
- CardSpec 阻塞项：cardspec 为 JSON 对象；suggestSize=2x2 与 DSL 一致；静态卡片未虚构 dataBindings / refreshPlan；UI 绑定路径可从初始 DataModel 推导。全部通过。
- 受保护文本换行评审：状态/时长/CTA 均单行完整显示，无 ellipsis/clip/marquee；action pill 为全宽非窄固定宽。通过。
- 设计评审（design-review.md）：紧凑、布局、视觉、交互、数据五项检查均通过。

## 宿主假设说明

- 静态卡片：能力清单仅声明 calendar / weather 两类能力，无睡眠数据能力。按 cardspec 规则不虚构能力，降级为静态卡片（CardSpec 仅含 suggestSize）。睡眠数据由初始 DataModel 提供演示值。
- 自定义动作函数 `openSleepReminderSetting` 为**宿主假设函数**，需宿主 catalog 已注册（用于打开早睡提醒设置）。

## 输出文件

- `card.genui`：genui DSL JSONL（3 行：createSurface / updateComponents / updateDataModel）。
- `card.cardspec.json`：CardSpec JSON 对象（静态，suggestSize=2x2）。
- `notes.md`：本记录。
