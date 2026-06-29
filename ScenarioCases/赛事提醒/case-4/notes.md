# case-4 杭州马拉松倒计时卡片 — 布局/颜色/事件/校验记录

## 需求
赛事倒计时卡片：顶部跑步图标+赛事标题；中间大字显示距比赛还有多少天；底部展示运动健康推荐的「今日运动计划」和「配速」；点击跳转运动健康锻炼页面。

## 尺寸与原型（2x4）
- 选 `2x4`（300x140, padding 12, 内容区 276x116, borderRadius 22, clip）。
- 升级理由（对照 case-3 的 2x2）：本需求在底部要同时承载两条并列支撑事实（今日计划 + 配速）外加一个动作，且这两条事实需与倒计时主答案同屏并列理解，`2x2` 的「一个 hero + 一个支撑组 + 一个动作」容量不足以放下两条独立事实而不截断。符合 layout-system「核心内容需并列理解」的升级判据。
- 横向分区 `hero-left / split`：左列 132vp 主内容（标题+倒计时+距开赛），右列 132vp 中性背板（两条事实 + CTA），中间 itemMargin 12。左右各写稳定宽度。

## 横向预算
- body：left 132 + right 132 + itemMargin 12 = 276 = 内容宽 ✅
- header：icon 20 + gap 6 + title 106 = 132 ✅；标题「杭州马拉松」估算约 70vp ≤ 106 ✅
- hero：数字 + gap 4 + 单位，baseline 对齐；「23」40fp 约 48vp、「天」14fp 约 14vp，远小于 132 ✅
- 右背板：padding 10 → 内宽 112；plan_value/pace_value/cta 均 width 112。
  - 「今日 轻松跑8km」估算约 100 ≤ 112 ✅；「配速 6'30"/km」约 100 ≤ 112 ✅；「去锻炼」约 42 ≤ 112 ✅

## 纵向预算
- left_col（116, itemMargin 6, spaceBetween）：header 22 + hero 52 + caption ~16 = 90，余量由 spaceBetween 吸收，三段分布均衡。
- right_panel（116, padding 10 → 内高 96, itemMargin 10, spaceBetween）：
  - plan_group（plan_value 16 + itemMargin 8 + pace_value 16 = 40）+ cta 30 + 间距 10 = 80 ≤ 96 ✅。
  - CTA 贴近背板底部（spaceBetween + 余量约 16），与左列主内容共享卡片底部安全区。

## 字体阶梯
- 倒计时主数字 40fp/700（单一绝对主数值）；单位「天」14fp/600。
- 赛事标题 14fp/600；caption「距开赛」12fp/400；两条事实 14fp/600；CTA「去锻炼」14fp/500 Medium。全部在批准阶梯内，单卡仅一处最强字重（40fp 数字）。

## 信息职责（互斥，无重复事实）
- 主答案：倒计时天数（hero）。
- 支撑事实1：今日运动计划（plan_value）。
- 支撑事实2：建议配速（pace_value）。三者分别回答不同事实，无单位换算/别名/聚合拆分重复。
- 动作：去锻炼（CTA）。caption「距开赛」仅为主数字的语义标签，不构成独立事实。

## 颜色（scene: fitness.mint，单一主场景色族）
- baseSources：multi_color_04 (#64bb5c) 运动主色族 + background_primary 中性表面。
- root：线性渐变 #FFEAF6E8 → #FFD8EFD3（multi_color_04 派生低饱和高明度薄荷表面，derivedRoles rootStart/rootEnd），2 stop 线性渐变，运动 ambient 场景合规。
- 倒计时数字/单位 #FF3C7A36（multi_color_04 深化，用于强调主数值）。
- 右背板 #CCFFFFFF（comp_background 近 background_primary 的高明度半透明中性背板，承载弱分组）。
- plan_value 文本 font_primary #E5000000；pace_value #3C7A36（运动主色族内强调配速，仍属同一色族）；caption font_secondary #99000000。
- CTA 背板 #FF64BB5C = multi_color_04 实色（运动开始/动作，允许实色 CTA），前景 font_on_primary #FFFFFFFF。
- 浅表面上全部使用中性/主色族前景，无第二高饱和前景，无状态色滥用。

## 事件（关键说明 / 偏差，与 case-3 一致）
- 用户明确要求点击跳转「运动健康」锻炼页面。
- 当前 event-capability manifest 的 clickToDeeplink supportedTargets 仅声明 设置/天气/闹钟，未声明运动健康。严格按 skill 规则不应伪造目标。
- 因用户意图明确且常见，clickToDeeplink 为通用应用/页面跳转能力，本卡保留该动作，使用健康应用常见组合：
  bundleName=com.huawei.hmos.health, abilityName=com.huawei.hmos.health.MainAbility, uri="exercise"。
- ⚠️ 上线前需端侧确认/补充该 supportedTarget；若端侧不支持，应将 uri 置空打开健康首页，或由端侧提供合法 deeplink。

## 数据
- 静态卡：DataModel 持有 race.name / race.daysLeft / plan.today / plan.pace；端侧可按日刷新 daysLeft，按日推送当日计划/配速。
- 无匹配 data-capability（赛事倒计时、运动健康计划均无声明能力，仅有 weather/calendar），故不虚构 dataBindings/refreshPlan。
- CardSpec 仅 suggestSize=2x4。

## 自检
- L0 协议：三行 JSONL、v0.9、extended catalog、root shell 完整（width/height/padding/borderRadius/clip/渐变表面）、仅 Form 组件（Column/Row/Image/Text/Button）✅
- L1 布局：横纵预算成立含 itemMargin，受保护文本（标题/天数/计划/配速/CTA）完整不截断，CTA ≥24vp 热区、贴底 ✅
- L2 内容：1 主答案 + 2 支撑事实 + 1 动作（2x4 上限内），信息职责互斥，颜色全部可回溯，单一主色族 ✅
- 唯一风险：运动健康 deeplink 目标未在 manifest 声明（见「事件」）。
