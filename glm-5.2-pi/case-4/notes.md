# Case-4 · Free Clip 2 耳机桌面卡片

## 用户需求
好看的耳机桌面卡片：显示 `Free Clip 2`、左右耳机电量（左 47%、右 95%），下方放「每日30首」「心动歌单」两个快捷入口；简洁高级，磨砂玻璃质感。

## 场景与模式
- 场景分类：`device/product`（耳机 + 电量）+ `action card`（两个快捷入口）。
- 模式：**模式 1（一句话桌面卡片）**，未提供已有 DSL。

## 尺寸选择：`2x4 / 320 x 160vp`
- 左/右耳机电量是天然左右关系；两个快捷入口也并列横排 → 需要横向宽度。
- `2x2 (160vp)` 同时容纳「设备名 + 左右电量环 + 两个入口」会拥挤，受保护文本不安全。
- `2x4` 给磨砂玻璃质感留出呼吸空间，主区域 = 3（header / 电量 / 入口），≤ 4，符合横版预算。

## 语义角色
| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | `Free Clip 2` | `nameText` |
| status | `已连接`（设备卡合理推断） | `statusChip`（绿点+文字） |
| primaryAnswer / metric | 左 47% / 右 95% | 双 `Progress` 环 + 叠加百分比 |
| action | 每日30首 / 心动歌单 | 两个磨砂玻璃可点击 `Stack` pill |

## 布局理由
- Root：`Stack` 320×160，圆角 22、`clip`，**深紫→靛蓝渐变**作为「玻璃背后的彩色底」，外加柔和阴影。
- 磨砂玻璃质感 = 渐变底 + 半透明白色块（`#24FFFFFF` 状态芯片、`#1FFFFFFF` 入口 pill）配 `#33FFFFFF` 细边框；Form 子集不支持真实模糊，用半透明叠层模拟。
- 主视觉焦点：左右两个电量环（`Progress ring`，单一青色强调 `#67E8F9`），环心叠加百分比；中间细分隔线区分左右。
- `mainCol` 用 `justifyContent: spaceBetween`，无固定 itemMargin，剩余高度自动均分到两段间隙。

### 宽度预算（296vp 内宽）
- headerRow（spaceBetween）：`Free Clip 2` ≈ 94vp + 状态芯片 ≈ 63vp ≈ 157vp ≤ 296 ✓
- earsRow（spaceEvenly）：左环 48 + 分隔线 1 + 右环 48 = 97vp，居中 ✓
- actionsRow（spaceBetween，itemMargin 10）：每 pill = (296−10)/2 = 143vp；内宽 127vp；「每日30首」「心动歌单」≈ 56vp ≤ 127 ✓

### 高度预算（136vp 内高，spaceBetween）
- headerRow ≈ 24vp；earsRow ≈ 65vp（环 48 + 4 + 标签 13）；actionsRow ≈ 32vp
- 合计 121vp，剩余 15vp → 两段间隙 ≈ 7.5vp，无溢出 ✓

## 受保护文本检查（无任何 ellipsis/clip/marquee）
- `Free Clip 2`、`已连接`：headerRow 宽裕，单行完整 ✓
- `47%`/`95%`/`左耳`/`右耳`：48vp 环/列内居中，放得下 ✓
- `每日30首`/`心动歌单`：143vp pill 内放得下 ✓

## 显式改进（v1 → 最终）
1. **v1 用厚 padding 的「玻璃面板」包住双环** → 高度超预算约 16vp。改为双环直接落在渐变上，环心叠加百分比，回收高度。
2. **v1 把 L/R 标签独立成行** → 拉高 earsRow。改为环下仅一行极小标签，收紧 itemMargin。
3. **v1 想用 `Button` 做磨砂入口** → Form 子集 `Button` 不支持 `backgroundColor`/`borderRadius`，无法做玻璃质感。改为 `Stack` + `onClick` 的可点击玻璃 pill（视觉交互指南明确推荐整区域可点击用 Stack/Row/Column onClick）。
4. **v1 考虑左环琥珀/右环青色双色** → 与「简洁高级」冲突。改为单一青色强调，电量差异由环填充度自显，调色更克制高级。
5. 尺寸由 `2x2` 升级为 `2x4`，确保左右电量关系 + 双入口 + 受保护文本全部安全。

## 半透明块说明（设计取舍）
- 主要半透明块控制在层级清晰：电量环为单一焦点；两个入口 pill 同属一个「动作区」（视觉从属）；状态芯片为极小徽章。三者层级分明，非互相竞争的小卡片，符合「外层单一卡片 shell」要求。

## CardSpec 决策：静态卡片
- 用户已给定具体电量值（47%/95%），且本 skill 能力清单仅声明 `calendar`/`weather`，**无 `device.battery` 能力**。
- 遵循「不虚构能力」约束 → 输出静态卡片：`{"suggestSize":"2x4"}`，无 `dataBindings`/`refreshPlan`。
- 电量值写入初始 `updateDataModel`，UI 通过 `$__dataModel.battery.*` 绑定。
- 如需实时电量，需端侧补充 `device.battery` capability manifest，再升级为动态 `dataBindings`。

## 宿主假设函数（需端侧 catalog 声明）
- `openPlaylist`：打开歌单，参数 `playlistId`（从 DataModel 绑定，未硬编码业务 ID）。
  - `playlist_daily30`、`playlist_heartbeat` 为示意 targetId，真实 ID 由宿主提供。

## DataModel 形状
```json
{
  "device": { "name": "Free Clip 2", "statusText": "已连接" },
  "battery": { "left": 47, "right": 95, "leftLabel": "47%", "rightLabel": "95%" },
  "actions": {
    "daily":     { "label": "每日30首", "targetId": "playlist_daily30" },
    "heartbeat": { "label": "心动歌单", "targetId": "playlist_heartbeat" }
  }
}
```

## 校验日志
```
=== GenUI 校验 (validate_genui_card.py) ===
GenUI 卡片校验通过
消息数：3
组件数：24
surfaceId：freeclip2-card

=== CardSpec 校验 (validate_cardspec.py) ===
CardSpec validation passed
```
- 错误：0；警告：0。
- 最终验收（review-validation + design-review）：协议/卡片/CardSpec 阻塞项全部通过；受保护文本无截断；单一焦点；层级清晰。

## 文件
- `card.genui.jsonl` — A2UI JSONL（createSurface / updateComponents / updateDataModel）
- `card.cardspec.json` — CardSpec（静态，suggestSize 2x4）
- `notes.md` — 本文件
