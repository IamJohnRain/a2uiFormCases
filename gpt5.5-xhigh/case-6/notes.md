# Layout Notes

- 场景/模式：time/reminder，模式 1 新卡片。
- 尺寸：选择 2x4。用户要求同时突出 32 天倒计时和显示今日计划，横版能给“大湾区城市马拉松”和“8km配速慢跑”足够保护宽度。
- 语义角色：identity=大湾区城市马拉松；primaryAnswer=32天；metric=倒计时天数；context=今日运动计划；status=夜跑节奏提示；media=无真实本地资源，使用渐变、半透明块和 RUN 字形；action=无显式交互，保持静态。
- 主区域：左侧倒计时焦点、右侧赛事标题、训练计划块、节奏提示，共 4 个以内。
- 视觉焦点：左侧半透明块内大号 32 + 天，背景深紫渐变与低透明 RUN 字形营造夜跑氛围。
- 信息节奏：先看倒计时，再看赛事名，再扫今日计划，最后查看状态提示。
- 关键横向关系：root 内容宽度 296vp；左焦点 122vp，gap 10vp，右侧约 164vp。右侧赛事名和计划值均独立成行，避免关键文本同挤一行。
- 拥挤 Row 预算：days_row 在左块内容宽约 102vp，数字 32 约 60vp，gap 4vp，单位“天”约 18vp，剩余约 20vp，安全；status_row 在右侧约 164vp，图标约 14vp，gap 5vp，状态文本可用约 145vp，状态为次要短文本。
- DataModel：按 event/countdown/training/visual 语义分组，所有展示字段预计算。
- CardSpec：静态卡片，suggestSize=2x4；未声明 dataBindings、writeResultTo 或 refreshPlan。
- 显式改进：初版考虑将“距离比赛还有32天”放在一行，但该行在 2x4 左块会挤压；改为“32 + 天”大号行，下方 caption 显示“距离比赛还有”，提升主指标完整显示安全性。
