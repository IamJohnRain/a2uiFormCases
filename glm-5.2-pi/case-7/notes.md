# case-7 · 会议提醒卡片（咖啡暖色调 + 专注模式）

一句话需求：会议提醒桌面卡片，咖啡色渐变暖色调；顶部“下一个日程”/“Agent需求评审会”；中间大号“14:00-15:30”+“还有15分钟后开启”；底部“专注模式”按钮（月亮图标）点击开启勿扰；半透明 + 柔和光影。

## 场景与模式

- 场景分类：`time/reminder`（会议 + 倒计时 + 动作）。
- 模式：模式 1（一句话新卡片，无既有 DSL）。
- 用户提供了具体内容（标题、时间、倒计时），未提供本地图片。

## 尺寸选择：2x2（160×160vp）

理由：

- 单条“下一个会议”提醒，竖向节奏：顶部标签 → 中间时间 → 底部动作。
- 主区域 = 3（identity / primaryAnswer / action），≤ 2x2 上限 3。
- 无左右关系、无媒体，不需要 2x4 横向宽度。
- 与场景映射“会议/提醒 → 2x2”一致。

## 语义角色

| 角色 | 内容 | 组件 |
| --- | --- | --- |
| identity | “下一个日程” + “Agent需求评审会” | 小 Text + 标题 Text |
| primaryAnswer | “14:00-15:30”（22fp，视觉锚点） | 大号 Text |
| context | “还有15分钟后开启” | 暖金色 Text（带光点） |
| action | “专注模式”（月亮图标） | 半透明玻璃 pill + onClick |

## 卡片形态：静态

- 用户显式给出了具体内容与相对倒计时“还有15分钟后开启”。
- Form 表达式不支持日期运算，`calendar` 能力 outputSchema（title/timeText/startAt/endAt/location）也不提供倒计时文本，无法动态产出“15 分钟后开启”。
- 依据第一原则（用户显式内容优先），采用静态展示卡，忠实呈现用户指定的全部内容。
- CardSpec 仅 `{"suggestSize":"2x2"}`，不虚构 dataBindings/refreshPlan。

## 视觉语言（为场景重选，非继承示例）

- 三段咖啡渐变（direction Bottom，光自上而下变深）：
  - `#8E5C3A`（暖咖啡）→ `#6E4528`（深咖啡）→ `#4A2E1A`（浓缩咖啡）。
- 文本：暖白 `#FFF6EC` / `#FFF8F1`（在中等深度咖啡上高对比），眉标 `#F2DEC2`。
- 强调色：蜜金 `#F5DCA8`（倒计时 + 光点，营造“即将开始/柔光”）。
- 半透明：focusPill `#30FFFFFF` + 描边 `#5CFFFFFF`（玻璃质感）。
- 柔和光影：root shadow `#4A2E1A` radius20 offsetY6（柔和暖色阴影）。

## 布局节奏（Column root, justifyContent spaceBetween）

- identityZone（眉标 11fp + 标题 14fp）≈ 34vp
- primaryZone（时间 22fp + 倒计时行 12fp，itemMargin 5）≈ 47vp
- focusPill（padding 7×2 + 内容 ≈17vp）≈ 31vp
- 三区合计 ≈ 112vp，剩 ≈24vp 由 spaceBetween 分两段 ≈12vp，自然形成“顶部/中间/底部”。

## 受保护信息宽度预算（关键信息完整显示，全卡无 ellipsis/clip/marquee）

- root padding 12 → 内容宽 136vp。
- timeText “14:00-15:30” @22fp ≈ 125vp < 136vp，留余量。
- countdownRow：●(~5) + itemMargin4 + “还有15分钟后开启”(9字符≈98vp) ≈ 107vp < 136vp。
- focusPill：🌙(~16) + itemMargin6 + “专注模式”(≈48vp) ≈ 70vp，居中于 136vp 玻璃条。

## 交互与 DataModel

- focusPill（可点击 Row）`onClick` → `enterFocusMode`（**宿主假设函数**），`args.mode="doNotDisturb"`，`args.meetingId` 绑定 `$__dataModel.meeting.id`（不硬编码业务 ID）。
- DataModel 按语义分组：`meeting`（id/sectionLabel/title/timeText/countdownText）、`action`（label）；展示串预计算，避免脆弱表达式。

## 第一版缺陷与改进（至少一次显式改进）

第一版设想：

1. 时间用 26fp → “14:00-15:30” ≈ 149vp > 136vp，会截断。
2. 倒计时为纯文本，缺“柔光/即将开始”的视觉提示。
3. 按钮无玻璃质感，与“半透明/柔和光影”要求不符。
4. root 无阴影，缺层次。
5. 渐变顶部过浅，浅色眉标对比不足。

改进：

1. 时间降到 22fp（验证 ≈125vp 完整放下），并保留 maxLines:1；不放 ellipsis。
2. 倒计时改为蜜金 Text + 暖金光点“●”（小 Row），呼应“即将开始/柔光”。
3. focusPill 改为半透明玻璃条（`#30FFFFFF` + 描边 `#5CFFFFFF`），满足半透明质感；月亮字形 14fp 与标签 12fp 分离控字号，更高级。
4. root 增加柔和暖色阴影（radius20 offsetY6）。
5. 渐变顶部取 `#8E5C3A`（中深咖啡），保证暖白/蜜金文本在顶部也有足够对比。

## 用户需求优先例外（记录）

- 用户要求“按钮带月亮图标”。Form `Button` 组件仅有 `label`，无图标槽位，无法承载图标。
- 依据第一原则，采用带真实 `onClick` 的可点击 `Row`（focusPill）承载 🌙 + “专注模式”：这是 visual-interaction 明确支持的“可点击区域使用 onClick”做法，并非无事件的假按钮。
- 协议合法性不受影响（真实 EventHandler、合法组件/样式/事件）。

## 校验日志

```
$ python3 scripts/validate_genui_card.py glm-5.2-pi/case-7/genui.jsonl
GenUI 卡片校验通过
消息数：3
组件数：12
surfaceId：meeting-focus-card
exit=0

$ python3 scripts/validate_cardspec.py glm-5.2-pi/case-7/cardspec.json
CardSpec validation passed
exit=0
```

两次校验均 0 错误 0 警告，最终验收未改动内容，无需复跑。
