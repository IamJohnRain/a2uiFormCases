# case-4 防沉迷卡片 设计与校验记录

## 需求拆解
- 主答案：抖音今日使用进度（已用 vs 每日限额），超出部分橙色醒目标出。
- 支撑事实 2 条：① 已用时长（+限额参考）；② 超出多少分钟。
- 主动作 1 个：修改每日使用时长限制 → 跳转系统设置。
- 顶部标题：应用使用时长监控。

## 尺寸决策
- 选 **2x4 (300×140)**。理由：用户明确要求“横向进度条 + 旁边显示具体用了多久和超出多少分钟”，需要横向宽度承载进度条全宽 + 并排两条事实 + 顶部标题 + 底部按钮，2x2 (116 内容宽) 放不下并排事实与全宽进度条。
- 内容区：300-24=276 宽，140-24=116 高。

## 纵向预算（root Column, padding12, itemMargin10, spaceBetween）
- titleRow 20 + bar 14 + factRow 34 + cta 34 = 102；+ 3×itemMargin(10)=30 → 132 ≤ 116? 
  - 注意：itemMargin 在 spaceBetween 下作为最小间距，4 段 3 间隔。102 固定 + 116 可用 - 102 = 14vp 余量分配给 3 间隔（约 4-5vp/间隔），spaceBetween 自动撑开。实际渲染按 spaceBetween 顶贴顶、底贴底。
  - 修正：为避免溢出，固定高度合计 102 < 116，剩余 14vp 由 spaceBetween 在 3 个间隙均分，底部 cta 贴安全区底边。✅
- cta 高 34 ≥ 24vp 热区；底边贴安全区底（spaceBetween 末位）。✅

## 横向预算
- titleRow 276 = titleText 188 + statusTag 80 + itemMargin 8 = 276 ✅
- factRow 276 = usedCol 132 + overCol 120 + itemMargin 12 = 264 ≤ 276 ✅（留 12 右侧呼吸）
- 进度条 barTrack/barBase 全宽 276 ✅

## 进度条超出可视化
- DataModel: usedMinutes=105, limitMinutes=60, overMinutes=45。
- 底层 Progress(capsule): value=limit(60)/total=used(105) → 蓝色填充 60/105≈57%（限额内部分，brand 蓝 #ff0a59f7）。
- 上层 overFill 橙块: 276×(45/105)=118vp，靠右(alignContent:end) 叠加，表示超出限额部分（alert 橙 #ffed6f21）。
- 二者拼接成“蓝(限额内)+橙(超出)”的完整条。轨道底色 comp_background_secondary #0c000000。

## 事实等价类去重
- 进度条只做可视化，不复述数值。
- usedValue 承载“已用具体时长”（唯一主事实），usedLabel 弱提示带限额参考（10fp 弱）。
- overValue 承载“超出分钟数”（独立事实，差值有新判断意义）。
- statusTag “已超时”是状态判断，不与数值重复。

## 颜色来源
- root bg = comp_background_primary (light #ffffffff)
- 标题 font_primary #e5000000；弱提示 font_secondary #99000000
- 进度限额内 = brand/multi_color_08 #ff0a59f7；CTA bg 同 brand，前景 font_on_primary #ffffffff
- 超出/超时 = alert #ffed6f21（真实“超出限额”告警状态，仅服务该事实，未作整卡主题）
- 轨道底 = comp_background_secondary #0c000000
- 单一主场景色族(brand 蓝) + alert 状态色，合规。

## 点击能力（关键取舍）
- manifest 中 `clickToDeeplink` 的“设置”应用 supportedTargets 仅声明 `intelligent_scene_entry`（情景模式/专注模式），**没有“应用使用时长管理”页面 URI**。
- 按 click-event 规则：不得伪造 URI。选最贴近防沉迷意图的合法目标——系统设置·情景模式(专注模式)，用户可在此开启专注/免打扰以限制使用。
- 如端侧后续提供“应用时长管理”页面的合法 bundleName/abilityName/uri，应替换该 onClick.args。

## 静态/动态
- 当前为静态降级：data-capability 目录无“应用使用时长”能力，**不虚构 dataBindings/refreshPlan**。
- usage 数据为占位示例值，端侧接入真实使用时长后通过 updateDataModel 刷新 /usage。
- cardspec 仅 suggestSize:"2x4"。

## 校验
- L0 协议：3 行 JSONL（createSurface/updateComponents/updateDataModel），catalog extended，Form 组件子集，onClick 来自已声明 clickToDeeplink。✅
- 绑定：title/statusLabel/usage.* 均在 updateDataModel.value 中。✅
- 受保护文本完整（标题/状态/已用/超出/CTA），无 ellipsis。✅
