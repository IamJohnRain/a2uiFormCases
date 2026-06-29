# case-2 应用时长监控卡片 — 布局与校验记录

## 需求
应用时长监控卡片：进度条 + 超出值 + 按使用习惯自动推荐最佳限制时长。

## 取舍
- 主答案：今日已用 vs 限制时长（`1h48m / 限 1h30m`），进度条做可视化。
- 支撑事实 1：超出值（差值，仅在已超限时承担新判断）→ `超出 18分钟`，alert 橙色。
- 支撑事实 2（派生新判断）：按近 7 天习惯推荐的最佳限制时长 `1h15m`。
- 主动作：去系统设置开启专注/情景模式（clickToDeeplink → settings intelligent_scene_entry）。
- 信息职责互斥：进度条只做可视化；`used_text` 同时承载已用(实际值)与限制(设定值)，不再单独复述两端值；`over_text` 只承载差值。

## 尺寸：2x4
2x2 放不下「进度条 + 超出 + 推荐时长面板 + 动作」四组并需可读推荐文本，故升级 2x4。
左列主内容 160vp + 间距 12vp + 右列 104vp = 276vp 内容区。

## 预算校验
- 横向 root：160 + 12 + 104 = 276 = 内容区宽 ✓
- 左列纵向：20 + 8 + 24 + 8 + 8 + 8 + 18 = 94 ≤ 116 ✓
- title_row：app_name 74 + 6 + status_tag 52 = 132 ≤ 160 ✓
- 右列纵向：rec_panel 74 + 8 + btn 34 = 116 ≤ 116（贴底）✓
- rec_panel 内：padding16 + (16+22+14) + itemMargin4 = 72 ≤ 74；内宽 104-16=88 ≥ 84 ✓
- 按钮热区 34vp ≥ 24vp，与右列同宽 104vp，贴安全区底边 ✓

## 颜色来源
- root 背景 `#ffffffff` = comp_background_primary (light)
- 主文本 `#e5000000` = font_primary；次级 `#99000000` = font_secondary；弱提示 `#66000000` = font_tertiary
- 超限状态 / 进度色 `#ffed6f21` = alert（真实「已超限」状态，服务进度条、状态标签、超出文案同一语义）
- 状态标签前景 `#ffffffff` = font_on_primary（处于 alert 饱和底上）
- 推荐面板 `#19000000` = comp_background_secondary（中性弱背板）
- 进度条底 `#19000000` = comp_background_secondary
- CTA `#ff0a59f7` = brand，前景 `#ffffffff` = font_on_primary

## 数据契约
未声明应用用量 data-capability，故 cardspec 不虚构 dataBindings/refreshPlan，仅 suggestSize=2x4。
运行时由端侧通过 updateDataModel 写入 /app、/usage、/recommend；当前值为可推导初始结构（非真实隐私）。
展示值优先完整表达式 {{ ... }} 读取 DataModel。

## 校验结论
L0 协议 / L1 预算 / L2 信息职责 / 颜色来源 / 事件能力（clickToDeeplink + 合法 settings 目标）均通过。
