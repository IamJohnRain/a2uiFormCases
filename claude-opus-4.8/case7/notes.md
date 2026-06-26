# Case 7 — 会议提醒卡片（咖啡色暖色调）

## 布局理由

### 场景/模式
- 场景：`time/reminder` + 内嵌 `action card`（专注模式按钮）
- 模式：Mode 1（一句话新建卡片）
- 能力范围确认：完全在支持范围内

### 尺寸选择：2x2（160×160vp）
选 2x2 而非 2x4 的理由：
1. 只需回答一个一眼扫视问题："下一个会议是什么、几点、要不要专注？"
2. header-primary-action 节奏在 160vp 内完全容纳，不需要左右关系。
3. 横版 2x4 的优势（左右双面板）在此场景并无必要，且会让单一焦点的时间数字反而显小。

### 语义角色
| 角色 | 内容 | 组件 |
|------|------|------|
| identity | "下一个日程" 标签 + 会议名称 | Column(scheduleLabel + meetingTitle) |
| primaryAnswer / metric | "14:00-15:30" 大字 | Text(timeRange), fontSize 24 |
| context | "还有15分钟后开启" | Text(reminder), fontSize 11 |
| action | "☾ 专注模式" 按钮 | Row(focusIcon + focusLabel) with onClick |

### 视觉焦点
唯一主导焦点：fontSize 24 的时间段 "14:00-15:30"，位于半透明 frosted 块内，深色渐变背景衬托。

### 信息节奏
```
header 区（~38vp）: scheduleLabel + meetingTitle（竖向紧凑，labelSmall + 14fp title）
timeBlock 区（~62vp）: 24fp 时间 + 11fp 提醒（frosted 半透明块，padding 10，borderRadius 16）
action 区（~34vp）: ☾ 专注模式 pill（中心对齐，padding 8，borderRadius 18）
```
三个区域通过 `Column.justifyContent: spaceBetween` + `itemMargin: 10` 分配 160 高度。
Root padding 12 → 可用内容高度 ≈ 136vp。符合 header-primary-action 节奏（38+62+34 = 134vp，≤ 136vp）。

### 宽度预算
| 区域 | 计算 | 结论 |
|------|------|------|
| meetingTitle "Agent需求评审会" | 160 - 24 padding = 136vp 可用；8 chars × 14fp ≈ 112vp | 安全，完整显示 |
| timeRange "14:00-15:30" | timeBlock: 160 - 24 - 20 block padding = 116vp；9 chars × 24fp ≈ 88vp | 安全，完整显示 |
| reminder "还有15分钟后开启" | 同上 116vp；8 chars × 11fp ≈ 88vp | 安全，完整显示 |
| focusAction "☾ 专注模式" | 136vp 可用；icon ≈ 18vp + gap 6 + label 4 chars × 13fp ≈ 52vp = 76vp | 安全，完整显示 |

所有受保护文本无需 ellipsis/clip，全部完整可见。

### DataModel 形状
按语义域分组：
```json
{
  "schedule": { "label", "title", "timeRange", "reminder" },
  "action": { "label", "icon", "meetingId" }
}
```
宿主动作参数 `meetingId` 从 DataModel 绑定，不硬编码。

### CardSpec
- suggestSize: "2x2"（与 DSL 一致）
- 静态卡片：用户提供固定内容，无动态日历能力需求，不虚构 dataBindings
- 所有 UI 绑定路径可从 updateDataModel 初始值推导

### 交互
- `focusAction` Row 带 `onClick: [{ "call": "enableFocusMode", "args": { "meetingId": "..." } }]`
- `enableFocusMode` 为宿主假设函数（打开系统勿扰模式）

---

## 改进说明

### 第一版内部草稿的不足
1. 渐变背景 + 纯文本竖排，缺少"半透明"层次感，不能体现用户要求的"柔和光影、高级感"。
2. 月亮图标直接用 Unicode 放进 Button.label，失去了图标+文字的并排视觉。
3. 动作区没有 pill 边框和光影，与背景融合，不够明确。

### 改进措施
1. 在 timeBlock 区加半透明 Column（backgroundColor: #1FFFFFFF，borderColor: #33FFFFFF，borderRadius: 16），形成 frosted glass 效果。
2. 月亮字形用独立 Text 组件（"☾"，fontSize: 15），与 focusLabel 并排在 Row 中，itemMargin: 6，中心对齐。
3. Root 加柔和 shadow（radius 16，#33000000），focusAction pill 加次级 shadow（radius 10，#2E000000），体现"柔和光影"。
4. 确认咖啡色渐变（#3A2A20 → #6B4A33 → #A9805C）为本场景新选，非从历史产物继承。

---

## 校验日志

### validate_genui_card.py
```
GenUI 卡片校验通过
消息数：3
组件数：10
surfaceId：meeting-focus-card
```
**结果：通过（0 错误，0 警告）**

### validate_cardspec.py
```
CardSpec validation passed
```
**结果：通过（0 错误，0 警告）**

---

## 最终验收结论

| 检查项 | 结论 |
|--------|------|
| 协议阻塞项（JSONL、version、catalogId、无 theme、单次 updateComponents、一致 surfaceId、无禁用组件/属性/事件） | 全部通过 ✓ |
| 卡片阻塞项（明确 2x2、主区域 ≤ 3、无滚动/列表/页面、规则构造、有布局理由和改进、可点击区域有 onClick） | 全部通过 ✓ |
| CardSpec 阻塞项（suggestSize 一致、静态无虚构 dataBindings） | 全部通过 ✓ |
| 受保护文本换行评审（时间、提醒、标题、CTA 均完整显示，无 ellipsis/clip） | 全部通过 ✓ |
| 设计评审（色板贴合场景、视觉焦点单一、半透明块有边框对比、交互真实、DataModel 语义化） | 全部通过 ✓ |
| 脚本校验（两个脚本均 0 错误 0 警告） | 全部通过 ✓ |

**卡片可交付。**
