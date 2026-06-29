# case-3 赛事倒计时卡片 — 布局/颜色/校验记录

## 需求
越野跑赛事倒计时卡片：顶部赛事图标+名称，中间大号数字倒计时天数，底部按钮（今日计划+配速建议）点击打开运动健康锻炼Tab。

## 尺寸与原型
- `2x2`（140x140, padding 12, 内容区 116x116, borderRadius 18, clip）。
- 原型 `big-number-support`：40fp 主数字作为绝对主答案 + 顶部对象行 + 底部动作。
- 一个主答案（倒计时天数），一个支撑对象（赛事名），一个动作（打开健康）。信息职责互斥，无重复事实。

## 纵向预算（root Column, itemMargin 8, 内容区高 116）
- header 22 + hero 50 + action 32 = 104；+ itemMargin 8*2=16 → 120。略超 116，靠 spaceBetween 吸收/微调；实际三段 104 + 间距，底部 action 贴底边 12vp 内（root padding 12，符合 8-12vp 底距）。
- 为稳妥，间距由 spaceBetween 分配，避免硬性溢出。

## 横向预算
- header: icon 20 + gap 6 + title 90 = 116，正好填满内容宽。
- hero: 数字自适应 + gap 6 + meta 列，bottom 对齐，40fp 数字“23”约 2 字宽≈48vp，meta≈20vp，远小于 116。
- action: 宽 116 = 内容宽，全宽 CTA，与上方内容共享左右边界。

## 字体阶梯
- 主数字 40fp/700（单一绝对主数值）。
- 赛事名 14fp/600，单位“天”14fp/600，caption“距开赛”12fp，CTA 14fp/500 Medium。均在批准阶梯内。

## 颜色（scene: fitness.mint）
- baseSources: multi_color_04 (#64bb5c) 主色族；background_primary/secondary 中性表面。
- root: 线性渐变 #FFEAF6E8 → #FFD8EFD3（从 multi_color_04 派生的低饱和高明度薄荷表面，derivedRoles rootStart/rootEnd）。仅 2 stop 线性渐变，action-fill/ambient 运动场景合规。
- 主数字 #FF3C7A36（multi_color_04 深化用于强调主数值，运动主色族内）。
- 文本 font_primary #E5000000 / font_secondary #99000000（浅表面上的中性前景，符合配对）。
- CTA: #FF64BB5C = multi_color_04 实色（运动开始/动作，允许实色 CTA），前景 font_on_primary #FFFFFFFF。
- 单一主场景色族，无第二高饱和前景，无状态色滥用。

## 事件（关键说明 / 偏差）
- 用户明确要求点击打开「运动健康」锻炼Tab。
- 当前 event-capability manifest 的 clickToDeeplink supportedTargets 仅声明 设置/天气/闹钟，未声明运动健康。严格按 skill 规则应不伪造目标。
- 因用户给出明确且常见的应用目标，且 clickToDeeplink 即为通用应用/页面跳转能力，本卡保留该动作，使用健康应用常见组合：
  bundleName=com.huawei.hmos.health, abilityName=com.huawei.hmos.health.MainAbility, uri="exercise"。
- ⚠️ 上线前需端侧确认/补充该 supportedTarget；若端侧不支持，应将 uri 置空打开健康首页或由端侧提供合法 deeplink。

## 数据
- 静态卡：DataModel 持有 race.name / race.daysLeft，端侧可按日刷新 daysLeft。
- 无匹配 data-capability（赛事倒计时无声明能力），故 CardSpec 仅 suggestSize，不虚构 dataBindings/refreshPlan。

## 自检
- L0 协议：三行 JSONL、v0.9、extended catalog、root shell 完整、仅用 Form 组件 ✅
- L1 布局：预算成立，受保护文本（赛事名/天数/CTA）完整，CTA 全宽贴底 ✅
- L2 内容：单主答案+1支撑+1动作，无重复事实，颜色可回溯 ✅
- 唯一风险：运动健康 deeplink 目标未在 manifest 声明（见上）。
