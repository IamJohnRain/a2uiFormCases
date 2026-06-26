# Case4 耳机桌面卡片 — 生成记录

## 用户需求

给我做个好看的耳机桌面卡片，显示 Free Clip 2 和左右耳机电量（47%、95%），下面放每日30首和心动歌单两个快捷入口，简洁高级一点，有磨砂玻璃那种感觉。

## 场景 / 模式识别

- 场景：`device/product`（产品标识 + 左右耳机电量状态）叠加 `action card`（两个音乐快捷入口）。
- 模式：模式 1（一句话桌面卡片，新建卡片，无已有 DSL）。

## 尺寸选择：2x4（320 x 160vp）

理由：

- 存在天然的左右关系——左侧是产品/电量焦点面板，右侧是两个堆叠的快捷入口。
- 受保护文本较多：产品名 "Free Clip 2"、两个电量值 "47%"/"95%"、两个入口标签 "每日30首"/"心动歌单"。2x2 的 136vp 内宽难以让这些值都完整、舒适地显示。
- 采用 `media-state-action` / `focal-detail-action` 横向节奏：左焦点面板 + 右动作列。

## 语义角色

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | Free Clip 2 + 副标题"无线耳机" | Column + Text |
| metric（主） | 左 47% / 右 95%，各配 capsule Progress | Row + Progress + 受保护 Text |
| action | 每日30首、心动歌单 两个可点击入口 | 两个可点击 Column pill |

主区域数 = 4（identity、左电量行、右电量行同属左焦点簇 + 两个动作 pill），符合 2x4 ≤4。

## 宽度预算

- root 320vp，padding 12*2 → 内宽约 296vp；itemMargin 10。
- 左焦点面板 `layoutWeight:1` ≈ 166vp，内 padding 14*2 → 内容约 138vp。
  - 电量行：L/R 标签固定 14vp + itemMargin 7 + capsule 54vp + itemMargin 7 + 百分比文本(14fp,"95%"≈30vp) ≈ 112vp < 138vp，安全。
- 右动作列固定宽 110vp，pill 全宽 110vp，中文标签 4-5 字 @14fp 单行可容纳。

## 磨砂玻璃实现（无真实图片素材）

按表现力工具箱，无本地图片资源时使用渐变 + 半透明块：

- root 深蓝灰渐变 `#1B1F2A → #2A3140 → #3A4358`（RightBottom），营造高级暗调底。
- 焦点面板与 pill 使用带边框的半透明白块（`#1FFFFFFF` / `#26FFFFFF` + `#33FFFFFF` 边框）= 磨砂玻璃质感。
- 心动歌单 pill 使用暖粉色调（`#3DFF6FA8` + `#55FF8FBE` 边框）增加场景含义。
- 电量用 capsule Progress 作为视觉锚点：低电量 47% 用暖橙 `#FF8A5B`，健康 95% 用绿 `#6FE3A8`。

## 受保护文本完整显示

"Free Clip 2"、"47%"、"95%"、"每日30首"、"心动歌单" 全部 `maxLines:1`、`textOverflow:"none"`，未使用 ellipsis/clip/marquee，按内容宽度自然排布。

## 交互 / DataModel

- 两个 pill 各有真实 `onClick` → 宿主函数 `openPlaylist`，`playlistId` 从 DataModel 绑定（`daily-30` / `loved-songs`），未在事件处理器中硬编码业务 ID。
- DataModel 按语义分组：`device`、`battery`、`entries`，展示字符串（"47%"、"95%"）预计算。

## CardSpec

- 静态卡片：电量/设备状态不在已声明能力清单（仅 weather / calendar）中，禁止虚构 `device.battery` 能力，故降级为静态卡片。
- `suggestSize: "2x4"`，无 `dataBindings`、无 `refreshPlan`。所有 UI 绑定路径均可从初始 DataModel 推导。

## 显式改进（至少一次）

第一版内部草稿把两个电量读数和两个快捷入口连同标识全部塞进右侧一个拥挤竖列——纯竖向堆叠、缺少清晰焦点层级，且 L/R 电量值在窄行内有与 capsule 进度碰撞的风险。

改进：拆成清晰的双 pane 外壳——左磨砂面板承载 标识 + 两条电量行（焦点簇），右列只放两个动作 pill；为每只耳机加 capsule Progress 视觉锚点，并让低电量（47%）用暖色、健康（95%）用绿色，在不增加杂乱的前提下加入场景特定语义。

## 校验日志

GenUI 校验脚本 `scripts/validate_genui_card.py`：

```
GenUI 卡片校验通过
消息数：3
组件数：20
surfaceId：freeclip-card
```

CardSpec 校验脚本 `scripts/validate_cardspec.py`：

```
CardSpec validation passed
```

两个脚本均 0 错误 0 警告，无需修复。

## 最终验收结论

- 协议阻塞项：全部通过（JSONL、version v0.9、extended catalog、无 theme、单次 updateComponents、surfaceId 一致、root 存在、引用可解析、仅用 Form 10 组件、Text.content/Image 无/Button 无、仅 onClick、无 $__widthBreakpoint、无网络/SVG/占位图片）。
- 卡片阻塞项：明确选 2x4、主区域 4 个、无滚动/长列表、规则构造、有布局理由 + 显式改进、可点击区域均有 onClick。
- CardSpec 阻塞项：suggestSize 与 DSL 一致（2x4）、静态卡未虚构 dataBindings/refreshPlan、UI 路径可从初始 DataModel 推导。
- 设计评审（design-review）：紧凑/布局/视觉/交互/数据五项均通过。

最终交付：通过。
