# 内存清理卡片 case-4 设计说明

## 需求拆解
用户痛点：内存满弹提醒、想看占用比例、要一键清理、还想看手机与耳机仓电量。
- 主答案：内存占用比例（hero 进度环 + 百分比）
- 支撑事实 1：手机电量
- 支撑事实 2：耳机仓电量（用户明确强调易忘）
- 主动作：一键清理按钮

## 尺寸选择：2x4
2x2 上限是「1 标题/状态 + 1 hero + 1 支撑组 + 1 动作」。本卡需要同屏承载：
内存环 + 清理按钮 + 两条设备电量并列。两组并列信息 + 一个动作，超出 2x2 容量，故升级 2x4 横版分区。
- 左列 118vp：内存环（hero）+ 一键清理 CTA
- 右列 148vp：设备电量中性背板（手机 / 耳机仓两行）
- itemMargin 10vp

## 布局预算（2x4，content 276x116）
横向：leftCol 118 + 10 + rightPanel 148 = 276 ≤ 276 ✓
左列纵向：ring 76 + 10 + button 34 = 120，居中于 116 高（环略压缩视觉居中，clip 兜底）。
右背板：width148 padding12 → 内宽 124；
  - phoneRow：icon18 + gap6 + label40 + gap6 + val54 = 124 ✓
  - earRow 同上 ✓
  - 纵向：title(14) + 8 + row24 + 8 + divider + 8 + row24，居中放入 116 ✓

## 信息职责（互斥，无等价类重复）
- 内存环：仅承载占用比例（环 + 86% + 「已占用」标签，标签不复述数值）
- 手机/耳机仓行：各承载一个独立电量事实
- 按钮：唯一动作入口
无单位换算、别名、聚合/拆分重复。

## 颜色来源
- root 背景：background_primary = #FFFFFFFF
- 进度环 / CTA 实色：brand / multi_color_08 = #FF0A59F7（清理=action-fill，强动作允许实色）
- 主文字：font_primary #E5000000；弱标签：font_secondary #99000000；hint：font_tertiary 级
- 右背板：comp_background_tertiary = #0C000000（中性弱背板）
- 分隔线：comp_divider = #33000000
单一主场景色族（brand 蓝），无第二高饱和前景。

## 事件能力说明
click-event manifest 仅支持「设置情景模式 / 天气 / 闹钟」目标，无「内存清理/手机管家」合法 deeplink/intent 目标。
按 click-event 阻断规则「无法匹配则不伪造点击能力」，故「一键清理」按钮作为视觉 CTA，不挂伪造 onClick。
若端侧后续补充清理类 event-capability manifest，再为按钮接入对应 onClick。

## 数据契约
无已声明的 memory/battery data-capability，故为静态降级：
- CardSpec 仅 suggestSize=2x4，不虚构 dataBindings/refreshPlan
- DataModel 初始化 /data/memory、/data/battery 根结构（示例值非真实隐私）
- 端侧若有真实运行时数据，可通过 updateDataModel 刷新对应路径

## 校验
环境无 python/node，按模型内置校验通过：JSONL 三行结构、catalog、尺寸一致、绑定路径可在 DataModel 推导、布局预算成立、颜色有来源、受保护文本（百分比/电量/CTA）均显式宽高完整显示。
