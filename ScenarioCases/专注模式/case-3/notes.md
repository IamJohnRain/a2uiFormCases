# case-3 会议日程卡（专注模式）布局与决策记录

## 需求拆解
- 主答案：当前会议（标题 + 时间段）。
- 支撑事实 1：会议室实时占用情况（status）。
- 支撑事实 2：空调温度（acTemp）。
- 主动作：专注模式按钮。

## 关键决策
1. **尺寸 2x4**：一个主对象（会议）+ 两条并列支撑事实 + 一个动作，2x2 容量不足，需横版并列理解 → 升级 2x4。
2. **数据能力降级（静态卡）**：`data-capability/` 仅声明 weather、calendar，无“会议室占用”“空调温度”能力。按 cardspec.md 规则不编造能力，改为静态 DataModel 承载占用/温度，CardSpec 不写 `dataBindings`/`refreshPlan`。
3. **事件能力**：专注模式按钮使用已声明 `clickToDeeplink` → 设置 `intelligent_scene_entry`（情景/专注模式），为 manifest 中合法目标组合。
4. **信息职责互斥**：时间段=主指标；会议室 status=占用状态；空调=温度设定；标题=对象。无单位换算/同义重复。

## 布局预算（2x4，内容区 276×116）
- root: Row，padding 12，itemMargin 12。
- 左列 leftCol width 150：标题(16fp) + 时间(20fp 强调) + 会议室位置(12fp)，纵向居中。
- 竖向 Divider：height 92，色 `#33000000`(comp_divider)。
- 右列 rightCol width 102：
  - 横向自检：150 + 12(itemMargin) + 1(divider) + 12(itemMargin) + 102 + 12*2(padding) ≈ 301 ≈ 300，留 clip 兜底。
  - 两条 label/value Row（height 18）+ 专注按钮（102×34），itemMargin 8，纵向 18+8+18+8+34=86 < 116。
- 专注按钮：4 中文字 ≈ 56 + 内边距，102vp 充裕；热区 34vp > 24vp。

## 颜色来源
- root 背景 `#FFFFFFFF` = background_primary(light)。
- 标题 `#E5000000` = font_primary；位置/标签 `#99000000` = font_secondary。
- 时间强调 `#FF0A59F7` = font_emphasize/brand（专注主色族）。
- 会议室“空闲”`#FF64BB5C` = confirm（真实可用状态）。
- 按钮 `#FF0A59F7`=brand 填充 + `#FFFFFFFF`=font_on_primary，单一主场景色族（蓝）。
- Divider `#33000000` = comp_divider。

## 模型内置校验
- L0：三行 JSONL，version/catalog/尺寸一致，root 引用存在组件并承载 shell。√
- 绑定：所有表达式路径（meeting.title/timeRange/room、room.status/acTemp）均在 updateDataModel.value 中。√
- 布局：横纵预算均成立，无截断，受保护文本（标题/时间/CTA）完整。√
- 颜色：全部可回溯 token，单一主色族。√
- 事件：clickToDeeplink 为已声明能力 + 合法目标。√
