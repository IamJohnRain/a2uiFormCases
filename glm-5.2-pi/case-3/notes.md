# Case-3 · Free Clip 2 耳机桌面小组件（设备状态 + 双快捷入口）

> 用户一句话：做一个耳机桌面小组件，干净高级、磨砂玻璃质感、耳机作为背景；显示耳机名称 Free Clip 2、左右耳电量（左 47% / 右 95%），一眼看出是否连上、还能用多久；底部放两个快捷入口「每日30首」「心动歌单」点一下即可打开；整体浅灰色调，带磨砂玻璃与柔和光影。

## 1. 场景与模式

- 场景分类：`device/product`（设备/产品状态）+ `action card`（快捷入口）。
- 模式：`模式 1：一句话桌面卡片`（新建卡片，用户未提供已有 DSL）。

## 2. 尺寸选择：`2x4`（320 × 160vp 横版）

依据 `card-composition-rules.md` 场景映射：设备/产品状态在「媒体 + 状态」时优先 `2x4`。理由：

- 用户明确要求 **两个**快捷入口（每日30首 / 心动歌单），`2x2` 仅 ≤3 主区域且单一主动作，两个 CTA 会与电量挤在同一狭窄列。
- 左右耳电量是天然左右关系，需要并排展示。
- 耳机背景 + 名称 + 双电量 + 双入口在 `160vp` 宽内会拥挤；`320vp` 给受保护文本（Free Clip 2、47%、95%、两个歌单名）留出安全横向空间。

> 受限例外（用户需求优先，记录于此）：用户显式要求 2 个动作。按 `card-composition-rules.md`，`2x4` 允许「一个主动作 + 一个从属动作」，用户明确要求时允许两个等权入口放入底部 actionRow。两个 tile 视觉等权、置底、可完整显示，未与电量竞争主视觉。

## 3. 语义角色与主区域

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | 耳机名称 Free Clip 2 | `nameText` |
| status | 连接状态（已连接 + 绿点） | `statusBadge` / `statusDot` |
| metric（左） | 左耳 47%（低电量，琥珀） | `leftValue` + `leftBar` |
| metric（右） | 右耳 95%（充足，绿） | `rightValue` + `rightBar` |
| action | 每日30首 / 心动歌单 | `tileDaily` / `tileFav`（可点击 Row + onClick） |
| media（弱） | 耳机背景水印 | `bgGlyph`（大号低透明 🎧） |

主区域 3 个（header / batteryRow / actionRow），≤ 2x4 上限 4。唯一视觉焦点 = 双电量（大字 + 色条），动作从属置底。

## 4. 布局理由（构图 + 宽度预算）

- Root：`Stack`，width 320 / height 160，圆角 22，`clip`。
  - 背景层：浅灰 `linearGradient`（#F5F7FA → #E9EDF2 → #DDE3EA，RightBottom）+ 柔和阴影（radius 26 / alpha 0.15 / offsetY 7）→ 干净 + 柔和光影。
  - `bgGlyph`：大号 🎧（fontSize 128，fontColor #202B3A1A，alpha≈10%）居中作为「耳机背景」水印；其上叠 `contentLayer`（透明），两个电量 frosted 块半透明，形成「磨砂玻璃罩住耳机」质感。
- 内容层 `contentLayer`：`Column`，padding 12，`justifyContent: spaceBetween`，三段：header → batteryRow → actionRow。
- 仅 **2 个主要半透明块**（满足 design-review「2x4 最多 2 个主要半透明块」）：
  - `batteryBlock`：单个磨砂面板（bg #7DFFFFFF / 描边 #A6FFFFFF / 圆角 16 / padding 12），内含 `leftGroup` ─ 细分隔线 ─ `rightGroup`，左右耳并排呈现。
  - `actionBlock`：单个磨砂面板（bg #8CFFFFFF / 描边 #AFFFFFFF / 圆角 16 / padding 10），内含 `tileDaily` ─ 细分隔线 ─ `tileFav`，两侧各自 onClick。
  - 分隔线 `Divider`（#26000000，strokeWidth 1）提供轻量结构，不再增加 frosted 块。
- `statusRow`（绿点 + 已连接）改为**无底色**纯文本行，避免变成第 3 个半透明块。

宽度预算（关键，防受保护内容被挤，基于 v2 合并面板）：

- `batteryBlock`：可用 = 320 − 24(contentLayer padding) = 296；面板宽 296，padding 12 → 内 272；`leftGroup`(lw1) + 分隔线(1) + `rightGroup`(lw1) → 每组 ≈135vp。
  - leftTopRow：`左耳`(12fp≈24) + `47%`(21fp≈30)，spaceBetween，合计≈54 < 135 ✓。
  - rightTopRow：`右耳` + `95%`，同上 ✓。
- `actionBlock`：可用 296；面板宽 296，padding 10 → 内 276；`tileDaily`(lw1) + 分隔线(1) + `tileFav`(lw1) → 每 tile ≈137vp。
  - `♪`/`♥`(15fp≈15) + itemMargin 6 + `每日30首`/`心动歌单`(14fp≈56) ≈ 77 < 137 ✓。
- `headerRow`：可用 296；`Free Clip 2`(17fp≈100) 左 + statusRow(绿点10+margin4+`已连接`≈36 ≈ 50) 右，spaceBetween，合计≈150 < 296 ✓。

所有受保护文本（Free Clip 2、左耳、47%、右耳、95%、已连接、每日30首、心动歌单）均不依赖 ellipsis/clip，可完整显示。

## 5. 显式改进（写 DSL 前至少一次）

第一版（脑内草案）设想：`2x2`，全幅耳机 `Image` 作背景，电量用小号文字 + 两个全宽按钮。问题：

1. `2x2` ≤3 主区域 + 单一主动作，装不下「名称 + 左右电量 + 双入口」，且双 CTA 会与电量争抢。
2. 用户未提供真实本地图片路径，按协议不得编造网络/资源 URL；`Image` 不可用。
3. 纯文字电量「一眼看不出还能用多久」。

改进：

1. **升级 2x4**：电量与双入口获得安全横向空间（见宽度预算），并记录双入口为用户需求例外。
2. **耳机背景改为大号低透明 🎧 字形水印**（expressiveness-toolkit「文本字形图标」+「无真实本地图片时用字形/渐变/半透明块」），自包实现「磨砂玻璃 + 耳机背景」，不虚构资源路径。
3. **电量色码化**：左 47% 琥珀 #F6A609（低）、右 95% 绿 #2FB987（充足），配合大字百分比，真正实现「一眼知道还能用多久」。
4. **显式连接状态徽章**：绿点 + 「已连接」，满足「一眼知道有没有连上」（断开时宿主可改 statusLabel 与点色）。
5. **DataModel 预算展示串**：`valueText:"47%"` 而非拼表达式，绑定稳健、易本地化；动作 ID 从 DataModel 绑定，不硬编码。
6. **收口半透明块（design-review 改进）**：第一版用 2 电量块 + 2 入口块 = 4 个 frosted 块，违反「2x4 最多 2 个主要半透明块」且显零碎。改为左右电量并入**单个磨砂面板**（细分隔线）、两个入口并入**单个磨砂面板**（细分隔线、两侧各 onClick），连接状态去底色，主要半透明块恰好 2 个，更干净高级。

## 6. 交互与 DataModel 形状

- 仅 `onClick`。两个入口 tile 各绑定宿主函数 `openPlaylist`（**宿主假设**：catalog 已声明该函数），`playlistId` 从 DataModel 取（daily30 / favorite）。
- DataModel 按领域分组：`data.device`、`data.battery.{left,right}`、`data.actions.{daily,favorite}`。
- 电量为静态演示值（无已声明设备电量能力，见 CardSpec 说明）。

## 7. CardSpec 形态

- 静态卡片：`{"suggestSize":"2x4"}`。
- 原因：本 skill 能力清单仅声明 weather / calendar，无设备电量能力；按 `cardspec.md` 不得虚构 `dataBindings` / `refreshPlan`。真实耳机实时电量需端侧补充 `device.battery` 类 capability manifest 后再以 dataBindings 接入（届时 UI 绑定路径需落在该能力的 `writeResultTo + outputSchema` 下）。

## 8. 校验日志

见第 10 节（脚本通过后回填）。

## 9. 最终验收要点

- 协议：v0.9 / extended catalog / 一次 updateComponents / createSurface 无 theme / 仅 onClick ✓
- 组件：仅 Form 10 组件；扁平邻接表 + id 引用；root 存在 ✓
- 受保护文本：无 ellipsis/clip/marquee；全部有完整宽度预算 ✓
- 视觉：单一视觉焦点（双电量）；最多 1 个次级装置（耳机水印）；≤2 个主要半透明类型（电量块 + 入口 tile）✓
- 交互：两个 tile 均有真实 onClick EventHandler，参数从 DataModel 绑定 ✓
- CardSpec：suggestSize 与 DSL 一致；静态形态不虚构能力 ✓

## 10. 校验与验收记录（脚本输出）

### 10.1 GenUI 校验（v2 终版）

```
$ python3 .agents/skills/harmony-card-generation-v5/scripts/validate_genui_card.py glm-5.2-pi/case-3/card.genui.jsonl
GenUI 卡片校验通过
消息数：3
组件数：28
surfaceId：free-clip-card
```

### 10.2 CardSpec 校验（v2 终版）

```
$ python3 .agents/skills/harmony-card-generation-v5/scripts/validate_cardspec.py glm-5.2-pi/case-3/card.cardspec.json
CardSpec validation passed
```

### 10.3 最终验收（review-validation + design-review）

- 协议阻塞项：version v0.9 / catalog extended / createSurface 无 theme / 单次 updateComponents / surfaceId 一致 / 仅 Form 10 组件 / 仅 onClick / 无 $__widthBreakpoint|$__colorMode / 无网络或占位媒体 → 全部通过。
- 卡片阻塞项：尺寸 2x4 / 主区域 3 ≤ 4 / 无滚动长列表表格段落 / 规则驱动非模板 / 已写布局理由 + ≥1 次改进 / 可点击区域（tileDaily、tileFav）均有真实 onClick → 全部通过。
- CardSpec 阻塞项：suggestSize 与 DSL 一致（2x4）/ 静态卡片未虚构 dataBindings|refreshPlan / UI 绑定路径均可由初始 DataModel 推导 → 全部通过。
- 受保护文本：headerRow / leftTopRow / rightTopRow / 两入口 tile 均做宽度预算，无 ellipsis|clip|marquee，关键文本完整可见。
- design-review：单一 shell（外圆角 22 ≥ 内圆角 16）/ 阴影属 root / 行内 ≤3 子节点 / layoutWeight 用于弹性列 / 单一主导视觉锚点（双电量）+ 一个次级装置（耳机水印）/ 主要半透明块 = 2 / 两 tile onClick 参数从 DataModel 绑定 / DataModel 按领域分组且预计算展示串 → 全部通过。

> 校验与最终验收均通过，交付终版 v2。
