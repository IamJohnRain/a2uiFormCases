# 会议日程小组件 — case-2 布局与校验记录

## 场景与取舍
- 主答案：今日会议日程列表（动态日历）。
- 主对象：今日 N 项会议，每项承载「开始时间 + 标题」两条事实，互斥不重复。
- 主动作：点击单条日程 → 跳日历详情（clickToIntent / ViewCalendarEvent）。
- 计数 `N 项` 为聚合事实，与列表项不重复（一个是总量，一个是逐项内容）。

## 尺寸选择：2x4（300x140）
升级 2x4 的具体失败点：日程是多条并列条目，每条「时间 + 标题」需要可读宽度；2x2 内容区 116vp 宽放不下时间块 + 分隔 + 可读标题，且只能容纳 1~2 行。2x4 给标题约 203vp 可读宽度，可容纳 3 条。

## 硬预算（content 276 x 116，root padding 12）
纵向：header 22 + itemMargin 8 + list 86 = 116 ✓
- header 行高 22；list 86，space 6，每行高 24 → 3 行 = 24*3 + 6*2 = 84 ≤ 86 ✓
横向（item_tpl Row，width 276）：
- item_time 48 + itemMargin 8 + divider 2 + itemMargin 8 + item_title 203 = 269 ≤ 276 ✓
- 时间文本 "14:00" 5 字符 ×0.6×14 ≈ 42 ≤ 48 ✓（受保护，textOverflow:none）
header 行（spaceBetween，276）：title 160 + count 108 = 268 ≤ 276 ✓

## 受保护文本
- 标题「今日日程」16fp Bold、时间 14fp Medium、计数 12fp 均 textOverflow:none，完整显示。
- 日程标题为可变长用户内容，非协议受保护字段，允许 ellipsis（已给固定 203vp 宽度兜底）。

## 颜色来源（单一中性 + brand 强调）
- root bg = background_primary → #FFFFFFFF（paper-panel）
- 标题/标题文本 = font_primary → #E5000000
- 计数 = font_secondary → #99000000
- 时间强调 = font_emphasize / brand → #FF0A59F7
- 分隔线 = comp_divider → #33000000
单一主色族（中性 + brand），无状态色滥用。

## 协议 / 绑定校验
- genui 三行：createSurface / updateComponents / updateDataModel ✓
- version v0.9，catalogId extended，2x4 尺寸与 cardspec 一致，root borderRadius 22 + clip ✓
- 仅用 Column/Row/List/Text/Divider；List 模板对象仅 componentId + path ✓
- 模板项内相对路径 dtStart / title / entityId 来自 calendar outputSchema ✓
- onClick 用已声明 clickToIntent + ViewCalendarEvent，参数 entityId 相对路径 ✓
- 计数表达式用 size()（唯一允许内置函数）✓
- DataModel 初始化 /data/calendar/items 空数组 + /state/loading ✓

## CardSpec
- suggestSize 2x4；dataBindings = calendar.events.search，writeResultTo /data/calendar
- timeInterval 为今日起止毫秒占位；bindingId today_events 供 refreshPlan 引用（onInstall/onVisible/periodic 1800s）。
