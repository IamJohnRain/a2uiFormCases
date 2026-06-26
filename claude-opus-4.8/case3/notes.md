# Case3 — 耳机桌面小组件 (Free Clip 2)

## 1. 场景 / 模式 / 尺寸识别

- **场景**：`device/product`（耳机设备状态：名称、左右电量、连接状态）+ 两个动作快捷入口。
- **模式**：模式 1（一句话生成新卡片，用户未提供已有 DSL）。
- **尺寸**：`2x2` / `160 x 160vp`。
  - 理由：用户要求“干净高级、简洁的桌面小组件”，内容可压缩为 3 个主区域（identity + 双电量 + 双快捷入口），无需横向宽度即可一眼读完，符合 2x2 速览定位。主区域数 = 3，未超过 2x2 上限。

## 2. 语义角色分配

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | 耳机名称 "Free Clip 2" | 顶部 `Text`（标题）|
| status | "已连接"（暗示是否连上）| 顶部右侧小 `Text` chip |
| primaryAnswer / metric | 左耳 47% / 右耳 95% | 两个半面板，每个含 side 标签 + 大百分比 `Text` + 线性 `Progress` |
| action | 每日30首 / 心动歌单 | 两个可点击 `Column` pill，各带真实 `onClick` |
| media（替代）| 耳机背景 | 大号低透明度耳机字形 🎧 + 浅灰渐变 + 磨砂半透明面板 |

## 3. 布局理由（写 JSON 前）

- **Root**：`Stack`，160x160，borderRadius 22，clip，浅灰 `linearGradient`（#F4F5F7 → #DEE1E7）+ 柔和 root 阴影。用 Stack 以叠加“柔和光影 + 磨砂玻璃”层。
- **层叠顺序**：`bgGlow`（柔和白色光晕圆块）→ `budGlyph`（耳机字形作背景视觉锚点，#14 低透明度）→ `panel`（磨砂玻璃内容面板，#80FFFFFF 半透明 + #5CFFFFFF 高光描边 + borderRadius 18）。
- **panel 内容**（padding 12 → 内部宽约 136vp，spaceBetween 竖向节奏）：
  1. `headerRow`：title（layoutWeight 1，14fp，minFontSize 12）+ statusChip（9fp，绿色半透明底）。
  2. `batteryRow`：两个等宽半面板（各 layoutWeight 1）。每个 = 竖排 [side 10fp / percent 18fp / linear Progress 56x4]。左暖橙 #E0884B、右绿 #4C8C5A 区分左右。
  3. `shortcutRow`：两个等宽 pill（各 layoutWeight 1，height 28，borderRadius 14），居中标签 11fp，各带 `onClick`。
- **视觉焦点**：单一主导锚点为左右大号百分比 + 背景耳机字形；一个主磨砂半透明块，符合 2x2「最多 1 个主半透明块、1 个视觉锚点」。
- **信息节奏**：identity（小）→ 双电量（主）→ 双入口（动作）。次要文本（side 标签、status）均短于主指标。

## 4. 拥挤 Row 宽度预算

- panel 内容宽 ≈ 136vp（root 内 padding 12）。
- **batteryRow**：136 − itemMargin 10 = 126，二等分 ≈ 63vp/半面板。"47%" / "95%" @18fp ≈ 26vp，安全完整显示；Progress 固定 56vp 适配。
- **shortcutRow**：136 − itemMargin 8 = 128，二等分 = 64vp/pill。"每日30首"/"心动歌单"（4 字 @11fp ≈ 50vp）在 64vp pill 内居中完整显示。
- **headerRow**：title 用 layoutWeight 1 弹性占据剩余，statusChip 内容宽自适应，"Free Clip 2" 单行 + minFontSize 12 完整显示。
- 所有受保护文本（名称、47%/95%、状态、pill 标签）均 `textOverflow:none`，无 ellipsis/clip/marquee。

## 5. 交互与 DataModel 形状

- 两个 pill 各有真实 `onClick`：`call: "openPlaylist"`，`args.playlistId` 从 DataModel 绑定（daily-30 / favorite-mix），业务 ID 不硬编码。
- `openPlaylist` 为**宿主假设自定义函数**（catalog 未显式声明，记为宿主假设）。
- DataModel 按语义分组：`device`、`battery.left/right`、`shortcuts.daily/favorite`，预计算展示字符串（percentLabel "47%"，value 47 供 Progress）。
- 用户明确要求 2 个快捷入口，属第一原则用户需求优先：2x2 默认建议单动作，此处按用户需求放置 2 个并排从属动作，视觉上不与主指标竞争。

## 6. CardSpec 形态

- **静态卡片**：`{ "suggestSize": "2x2" }`。
- 耳机电量 / 歌单数据由宿主 App 提供；能力清单仅声明 calendar/weather，**无耳机电量能力**。按协议「不虚构能力」，本卡片降级为静态卡片，仅声明 suggestSize，不编造 dataBindings/refreshPlan。UI 绑定路径均可从初始 DataModel 推导。
- suggestSize 与 DSL 尺寸一致（2x2）。

## 7. 显式改进（至少一次）

- **第一版缺陷**：把左右电量做成单行 4 个文本碎片（左 47% 右 95%），受保护百分比有被拥挤/拆分风险，且左右层级不清；同时第一版把标签组与动作挤在一行。
- **改进**：
  1. 电量重构为两个平衡半面板，每面板含独立 side 标签 + 大百分比 + 微型线性 Progress，左右层级清晰、更有设备感。
  2. 两个快捷入口移入独立 `shortcutRow`，与指标行分离，避免「标签组 + 动作」同行争抢。
  3. 增加浅灰渐变 + 磨砂半透明面板 + 低透明度耳机字形，落实用户要求的「磨砂玻璃 + 柔和光影 + 浅灰高级感」。

## 8. 受限例外说明

- 用户要求「耳机作为背景图」，但未提供真实本地/资源图片路径。按协议「不得编造网络/占位 Image.src」，改用**大号低透明度耳机字形 🎧 + 浅灰渐变 + 磨砂半透明块**作为非媒体视觉替代，保留耳机背景观感而不违反媒体真实性约束。
- `openPlaylist` 标记为宿主假设自定义函数。

## 9. 校验日志

genui 第一版误用了 `paddingHorizontal`/`paddingVertical`（不在 validator 允许样式键内），改为 `padding:4` 后重校验。

```
GenUI 卡片校验通过
消息数：3
组件数：22
surfaceId：earbuds-card
---CARDSPEC---
CardSpec validation passed
```

两脚本均 0 错误 0 警告。

## 10. 最终验收结论

- 协议阻塞项：全部通过（validator 绿）。
- 卡片阻塞项：2x2 明确；3 主区域 ≤ 3；无滚动/长列表/段落；规则构造；有布局理由 + 1 次显式改进；两 pill 均有 onClick。
- CardSpec 阻塞项：present、object、suggestSize 一致、静态无虚构 bindings。
- 受保护文本换行评审：名称 / 47% / 95% / 状态 / pill 标签全部完整显示，无 ellipsis/clip/marquee。
- 设计评审（视觉/交互/数据）：单一主导锚点、磨砂半透明块带高光描边、左右配色区分、动作参数 DataModel 绑定、语义化数据键，均通过。
- 评审未修改内容，校验保持通过。**可交付。**
