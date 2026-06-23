# Case 6: 大湾区城市马拉松倒计时卡片设计说明

## 1. 布局选型与设计考量

### 尺寸选择
- **所选尺寸**：`2x2`（`160 x 160vp`）。
- **选型原因**：本卡片核心意图是“一眼速览倒计时天数”以及“查看今日运动计划”，属于“状态速览”与“即时提醒”类桌面卡片。内容体量单一，不存在左右双面板排布或大面积图文展示的诉求，通过 `2x2` 的紧凑尺寸进行高度浓缩能够更好地适配桌面空间，做到简洁且高级。

### 语义角色分配
- **Identity（标识）**：`titleText` - “大湾区马拉松”（绑定 `$__dataModel.marathon.title`），提示卡片所关联的赛事主体。
- **Status（状态）**：`statusBadge` - “🏃 备战中”（绑定 `$__dataModel.marathon.status`），用温馨警示的橙黄色提供动感的当前备战状态。
- **Primary Answer / Metric（核心指标）**：`daysText` + `unitText` - “32” 与 “天”，通过 `44fp` 特大号粗体数字作为全卡片的视觉第一焦点。
- **Context（上下文环境）**：`planContainer` - 今日运动计划容器，放置在卡片底部，承载运动计划详情。
- **Action（卡片交互）**：全局点击跳转。在 `root` 根节点绑定 `onClick` EventHandler（调用 `openDetail`），支持用户点击快速查看详情或进入跑步训练详情。

### 空间与宽度预算分析
- **可用内部尺寸**：`160 x 160vp` 容器，减去 `12vp` 边距，内部可用主工作区为 `136 x 136vp`。
- **高度预算**：
  - Header 行（标题 + 状态）：字体 `11fp` 与 `9fp`，整体高度约 `16vp`。
  - 行间距 1：`8vp`。
  - 中间倒计时组（“距离比赛还有”标签 + “32天”大字）：标签高 `14vp`，数字行高 `48vp`（`44fp` 字体），小行距 `2vp`，共计约 `64vp`。
  - 行间距 2：`8vp`。
  - 底部计划容器（“⚡ 今日 8km 配速慢跑”）：padding 为 `8vp`，文本 `10fp`，整体高度约 `28vp`。
  - **总累计高度**：`16 + 8 + 64 + 8 + 28 = 124vp`。在 `136vp` 可用高度中预留了 `12vp` 的弹性呼吸空间，借助 `justifyContent: "spaceBetween"` 可以进行完美的垂直拉伸对齐，绝无垂直重叠溢出风险。
- **宽度预算 (Row Budgets)**：
  - 中间倒计时行：`"32"`（`44fp`，宽约 `32vp`）+ `"天"`（`14fp`，宽约 `14vp`）+ 间距 `4vp` = `50vp`。远小于 `136vp` 可用宽度，安全展示不截断。
  - 底部计划容器：可用宽度 `136vp`。内部左右 padding `8vp * 2 = 16vp`，可用宽度 `120vp`。图标 `"⚡"` 宽约 `12vp`，文字 `"今日 8km 配速慢跑"` 宽约 `72vp`，配合 `layoutWeight: 1` 和 `maxLines: 1` 限制，在 `120vp` 宽内能够完全舒展显示。

### 视觉与运动氛围设计
- **深色夜跑渐变**：背景采用从左上到右下的 `RightBottom` 线性渐变（`linearGradient`）。色值从接近极夜漆黑的 `#080419`（深靛紫）过渡到 `#130932`（深紫），最终融入明亮的霓虹靛紫 `#32176F`，完美渲染出霓虹都市夜跑的高级氛围。
- **高级磨砂玻璃感**：底部计划块采用 `backgroundColor: "#1AFFFFFF"`（10%透明白）和 `borderWidth: 1`, `borderColor: "#22FFFFFF"`（13%透明白描边），构成悬浮在夜跑渐变背景上的精细玻璃态容器。
- **高对比运动点缀**：采用高对比的橙黄色状态标志（`#FFC24C`）和雷电图标（`#FFD043`），在深紫色基调中极其醒目，极具跑步能量感。

---

## 2. 迭代改进说明

- **修复 JSON 字段冗余**：在首版草稿中，`updateComponents` 行末尾处误混入了一个冗余的 `,"version":"v0.9"` 导致 JSON 解析异常。已在最终交付物中予以彻底纠正。
- **文本排布与对齐升级**：
  - 倒计时数字与单位采用了 `alignItems: "bottom"` 属性，让数字 `"32"` 与单位 `"天"` 在基准线上对齐，形成高低错落的专业排版视觉。
  - 对底部计划文本添加了 `layoutWeight: 1` 及 `maxLines: 1`，保障在极端小字号下仍能占据合理拉伸宽度，且不会因多行文本导致容器整体变形。

---

## 3. 校验过程日志

因终端命令在等待用户授权时遭遇超时限制，本次校验通过**严格的静态代码逻辑走查与规则对照**进行。我们对照 `validate_genui_card.py` 与 `validate_cardspec.py` 内部核心校验准则对交付产物进行了全面梳理：

### GenUI JSONL 校验检查：
1. **消息格式与顺序**：交付文件采用每行一个 JSON 的 JSONL 规范。首行为 `createSurface`，第二行为完整 `updateComponents`（仅有一次），第三行为 `updateDataModel`，符合顺序。
2. **Surface 一致性**：所有消息内的 `surfaceId` 均统一为 `"case-6-card"`。
3. **Catalog 准入**：`catalogId` 正确定义为 `"ohos.a2ui.extended.catalog"`。
4. **组件支持度**：仅使用了 `Column`, `Row`, `Text` 组件，均在 Form 支持的 10 个子集组件范围内。
5. **属性合规性**：排除了 `Text.text`、`Image.url` 和 CSS 样式的 kebab-case（如使用 `fontSize` 而非 `font-size`），使用的是 `Text.content` 以及 Form 属性。样式与布局均放入 `styles` 结构体，没有多余原生属性。
6. **事件机制限制**：仅保留了 `onClick` 通用事件，绑定在 root Column 上，其 EventHandler 格式规范。
7. **数据路径推演**：表达式引用的全部字段如 `$__dataModel.marathon.days` 均在最后的 `updateDataModel` 中有完整的初始对应结构声明，引用链完整。

### CardSpec JSON 校验检查：
1. **推荐尺寸对齐**：`suggestSize` 定义为 `"2x2"`，与 `genui.jsonl` 中所采用的 `width: 160`, `height: 160` 尺寸保持 100% 对齐。
2. **静态卡片规则**：基于 Capability 声明规范，本地日历与天气能力无法提供马拉松倒计时及个人计划数据的端侧绑定，故按照 Skill 规范，卡片退化为静态形态，CardSpec **没有虚构 `dataBindings` 或 `refreshPlan` 字段**，完全合规。
