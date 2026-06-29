# case-4 会议日程 / 专注模式卡片 — 布局与校验记录

## 需求拆解
- 一个主答案：下一个会议（名称 + 时间）。
- 支撑事实：提前多少分钟提醒（弱提示，10fp）。
- 一个主动作：开启手机专注模式（CTA 按钮）。
- 顶部品牌区：会议图标 + "会议日程" 标题。

## 尺寸与布局预算（2x2，140×140，padding 12，内容区 116×116）
Column itemMargin 4，5 个区块纵向排布：
- header  18 (Row：icon16 + gap4 + text96)
- title   20 (16fp medium，单行)
- time    18 (Row：icon14 + gap4 + text98，14fp)
- remind  14 (10fp 弱提示)
- cta     30 (Button，14fp medium)

纵向占用 = 18+20+18+14+30 = 100，itemMargin 4×4(间隔) = 16，合计 = 116 = 内容区高度，预算精确成立。
横向：header/time 行内 icon + gap4 + text，宽度均 ≤116，成立。
CTA 贴底：root padding 12 即为底边距，符合 2x2 底边距 8-16vp。

## 颜色来源
- root 背景 #ffffffff = background_primary(light)
- 标题 #99000000 = font_secondary
- 会议名 #e5000000 = font_primary
- 时间 #e5000000 = font_primary
- 提醒弱提示 #66000000 = font_tertiary
- CTA 背景 #ff0a59f7 = brand / background_emphasize(light)
- CTA 文字 #ffffffff = font_on_primary

## 事件能力
- CTA onClick = clickToDeeplink → 设置 com.huawei.hmos.settings / MainAbility / uri=intelligent_scene_entry
  （manifest 中 "打开系统设置中的情景模式，用户可以打开免打扰或专注模式"，语义匹配专注模式）。

## 数据能力（动态）
- calendar.events.search，timeInterval 为今日起止毫秒（占位），writeResultTo=/data/calendar。
- UI 读取 /data/calendar/items/0 的 title、dtStart、dtEnd、remindTime[0]。
- 初始 DataModel items=[]、state.loading=true，不写死隐私日程。

## 信息职责自检
- 会议名（对象）/ 时间段（时间点）/ 提醒（派生提醒判断）/ 动作入口，四者互斥，无重复事实。
- 时间用 "dtStart - dtEnd" 单一表达式承载，未在别处复述。
