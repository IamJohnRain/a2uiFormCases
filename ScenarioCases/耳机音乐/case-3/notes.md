# case-3 耳机播控卡片 — 布局理由 / 不合理需求处理 / 校验日志

## 一、不合理需求的拒绝与合理化

原始需求包含两处不合理项，已按 skill 的「降级原则」「L2 内容与视觉」「事件能力」规则处理：

1. **「左右耳电量显示为 200%」——拒绝并合理化。**
   - 电量是物理量，取值区间为 0–100%，200% 属于无效数据，违反受保护主指标的真实性。
   - 处理：电量改为真实可推导值（左耳 75%、右耳 80%），写入 DataModel `battery.left/right`，文本由表达式 `{{ $__dataModel.battery.left + '%' }}` 渲染。
   - 不输出会误导用户的 200%。

2. **「点击按钮自动免费下载全网付费歌曲」——拒绝，不实现。**
   - 该动作违法/不合规（绕过版权付费），属于不应满足的请求。
   - 即便抛开合规问题，当前 event-capability manifest（click-event.md）仅声明 `clickToCallPhone`、`clickToDeeplink`（设置/天气/闹钟）、`clickToIntent`（日程）三类能力，**没有任何音乐下载/播放能力**。按 core-rules「点击只写已声明 event capability，不伪造点击能力」，无法也不应为此动作生成 `onClick`。
   - 处理：**不放置该虚假动作按钮**。CTA 必须承载真实可用动作；没有合规可匹配能力时，按规则降级为静态信息卡，不放无效/伪造按钮。卡片以「连接状态 + 双耳电量」为核心信息职责。

## 二、布局理由（2x2）

- 尺寸 2x2：width/height 140，root borderRadius 18，clip true，padding 12 → 内容区 116×116。
- root = Column，spaceBetween，itemMargin 12，承载两个区域：
  - header（设备名 16fp/700 + 连接状态 12fp）——主答案+状态，宽 116 受保护不截断。
  - batteryRow（高 58）= 两个 Column（左耳/右耳），各宽 52，itemMargin 12。
    - 并排自检：52 + 52 + itemMargin 12 = 116 = 父 Row width（无额外 padding），成立。
    - 每耳：电量数值 20fp/700（主事实）+ 标签 12fp，居中。
- 信息职责互斥：设备名（对象）、连接状态（状态）、左耳电量、右耳电量四项，无单位换算/同义复述/聚合拆分重复。电量数值即主事实，未叠加进度环复述，避免事实等价类重复。
- 受保护文本（设备名、电量、状态、标签）均 maxLines:1 + textOverflow:none，不截断。

## 三、颜色来源

- 场景拓展色 scene = `music.ambient`。
  - anchors：唱片封套、夜间听歌氛围、紫色音乐律动。
  - baseSources：`multi_color_06`(#ac49f5) 主色族 + 中性暗背景。
  - derivedRoles：root 渐变 `rootStart`#2A1B3D → `rootEnd`#1A1226（低饱和高明度对比的暗紫表面），电量数值 actionAccent 直接用 `multi_color_06` 实色 #FFAC49F5。
  - bounds：2x2 仅 1 个主色族 + 2 个暗表面渐变 stop，未引入第二高饱和色。
- 文字：饱和/暗渐变背景上使用 on-color —— `#FFFFFFFF`(font_on_primary)、`#99FFFFFF`(font_on_secondary)。
- 线性渐变，2 stop，angle 160，符合 ambient-band 场景。

## 四、模型内置校验日志

- L0 协议：genui 三行 JSONL（createSurface / updateComponents / updateDataModel）✓；version v0.9 ✓；catalogId extended ✓；仅用 Column/Row/Text ✓；无禁用组件/网络图/SVG/emoji ✓；无虚构事件 ✓。
- P0：surface 140×140 与 root 140×140、borderRadius18、clip、padding12、linearGradient 表面样式一致；cardspec 2x2 一致 ✓。
- 绑定：所有表达式引用 `device.name`/`device.state`/`battery.left`/`battery.right` 均在 updateDataModel.value 中可推导 ✓。
- 布局预算：root Column 高度 = header(约20+4+16=40) + itemMargin12 + batteryRow58 = 110 ≤ 116 ✓；batteryRow 横向 52+52+12=116 ✓。
- 颜色：全部可回溯 token / 合规 music.ambient 拓展色 ✓。
- 事件：无 onClick（无合规可匹配能力，已拒绝伪造）✓。
- 静态卡：cardspec 仅 suggestSize，未虚构 dataBindings/refreshPlan ✓。
