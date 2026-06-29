# 睡眠卡片 case-2 设计与校验记录

## 取舍
- 主答案：昨晚睡眠时长（7h32m，20fp 主数值）。
- 支撑事实（2 条，职责互斥）：
  - 睡眠质量评分 → Progress ring（可视化承载，旁边不复述数值）。
  - 深睡时长 → 文本（与总时长不同事实，非派生重复）。
- 主动作（1 个）：设置闹钟 → clickToDeeplink 闹钟应用（supportedTargets 合法目标）。
- 静态卡片，无声明的睡眠 data capability，故 cardspec 只有 suggestSize，不虚构 dataBindings。

## 尺寸与布局（2x2）
- root Column 140x140，padding 12 → 内容区 116x116，borderRadius 18，clip。
- 原型：meter-plus-fact（ring + 事实）+ 底部动作。
- 纵向预算：header 18 + 8 + hero 54 + 8 + action 28 = 116 ≤ 116 ✓。
- hero Row 横向：ring 52 + itemMargin 12 + facts 52 = 116 ≤ 116 ✓。
- header Row：moon 14 + itemMargin 6 + title(12fp) 剩余 96 ✓。
- 动作区贴底：底边距 12（root padding），≤16 ✓，与 hero/header 同宽 116 共享左右边界。
- 受保护文本：标题、时长、深睡、CTA 均 maxLines:1 + textOverflow:none，宽度足够，不截断。

## 颜色（sleep.night 场景拓展，dark 取值）
- root 渐变：#ff1b1b2e → #ff2a2550（线性 angle160），夜间睡眠暗紫表面。
  - scene: sleep.night；anchors: 夜空、夜灯；baseSources: background_tertiary(#ff202224)、multi_color_01(dark #5f58c7)；derivedRoles: rootStart/rootEnd；低饱和高暗度。
- 进度环：#8981f7（multi_color_aux_01 light，深底上保证可读）。
- 前景文字：#e5ffffff(主)、#99ffffff(次)，on-color 中性前景，符合饱和/渐变背景前景规则。
- CTA 背板：#19ffffff（comp_background_secondary dark），中性磨砂层；CTA 文字 #e5ffffff。
- 单一主场景色族，无第二高饱和前景，无状态色滥用。

## 素材
- 头部图标 icon_sleep.png（asset-library 睡眠监督场景，真实声明路径），14x14 contain。

## 校验
- L0 协议：三行 JSONL（createSurface/updateComponents/updateDataModel），version v0.9，catalog extended，root 引用存在组件，shell 完整。组件全部在 Form 10 组件内。
- 绑定：所有 {{ $__dataModel.sleep.* }} 在 updateDataModel.value 中可推导。
- 事件：onClick 仅 DSL，clickToDeeplink 闹钟为合法 supportedTarget；cardspec 不含点击。
- 尺寸：DSL 与 cardspec 均 2x2 一致。
