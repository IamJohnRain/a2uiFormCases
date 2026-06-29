# 耳机音乐播控卡片 — case-1 notes

## 场景与取舍
- 场景：戴耳机播控（music ambient）。
- 主答案：当前播放曲目（歌名）。
- 支撑事实（≤2）：歌手、播放进度。
- 状态：顶部「正在播放」+ 耳机图标（识别场景）。
- 动作区：上一首 / 收藏 / 下一首 三个控制图标。

## 尺寸
- 选 2x2（140×140），内容区 116×116，root padding 12。
- 信息容量：状态行 + hero(歌名/歌手) + 进度 + 控制行，符合 2x2「一标题/状态 + 一 hero + 一支撑 + 一动作组」上限。

## 纵向预算（116vp 高，itemMargin 8，3 个间隔=24）
- topRow 20 + heroCol(16+4+12≈32) + progress 4 + actionRow 34 = 90
- 90 + 24(itemMargin) = 114 ≤ 116 ✓，justifyContent spaceBetween 吸收余量，底部控制行贴近安全区底边。

## 横向预算
- 所有主区域宽度固定 116，等于内容区宽度 ✓。
- actionRow：3×26 图标 + 2×18 itemMargin = 78 + 36 = 114 ≤ 116，居中 ✓。
- topRow：18 图标 + 6 gap + 「正在播放」4字×12fp≈48 = 72 ≤ 116 ✓。

## 受保护文本
- 歌名 16fp 1 行；歌手 12fp 1 行；状态 12fp。均 textOverflow:none，不截断。
- 示例数据「晴天」「周杰伦」在 116 宽内充分可放下。

## 颜色来源
- root 渐变：music.ambient 场景拓展，baseSources = multi_color_06(#ac49f5 / dark #8c55c2)。
  - rootStart #FF3A2A5C、rootEnd #FF1E1830：从紫色族派生的低饱和暗氛围表面（唱片封套/夜间聆听锚点）。
- 进度高亮 #FFC386F0 = multi_color_aux_06(light) 直取，作进度可视化主色。
- 文字：on-dark 白色族 #FFFFFFFF / #99FFFFFF（font_on_primary / font_on_secondary 角色）。
- 进度槽 #33FFFFFF、控制图标无第二高饱和前景，单一主场景色族 ✓。

## 事件能力决策
- 已声明 event-capability 仅 clickToCallPhone / clickToDeeplink(设置/天气/闹钟) / clickToIntent(ViewCalendarEvent)。
- 无音乐播控（上一首/下一首/收藏）或音乐 App 跳转目标。
- 按规则「意图无法匹配 supportedTargets 时不要伪造点击能力」，故控制元素用 Image 静态呈现，不写虚构 onClick。
- 因此为静态卡片，cardspec 只写 suggestSize，无 dataBindings / refreshPlan。

## 校验
- L0 协议：三行 JSONL、v0.9、extended.catalog、仅用 Form 组件（Column/Row/Image/Text/Progress）✓。
- L1 布局：纵横预算成立，受保护文本不截断，底部动作贴底 ✓。
- L2 信息职责互斥：歌名/歌手/进度/状态各承载唯一事实，无重复 ✓。
- 颜色全部可回溯 token / 场景拓展色 ✓。
- 注：技能目录未提供 scripts/validate_card.py，采用模型内置校验。
