# 专注模式 / case-5 — 会议日程卡片 notes

## 需求拆解
- 顶部：会议图标 + 标题
- 中间：下一个会议名称（大字）+ 具体时间
- 下方小字：提醒设置（提前 10 分钟提醒）
- 底部：专注模式按钮，点击进入免打扰/情景模式

## 信息职责（互斥，无重复）
- header.title = 卡片身份「会议日程」
- meeting.name = 主答案（下一个会议是什么）
- meeting.timeLabel = 时间点支撑事实
- meeting.reminderLabel = 提醒设置弱提示（10fp）
- action = 唯一主动作（进入专注模式）

## 尺寸与布局（2x2，140x140，padding 12，内容区 116x116）
root Column 用 `justifyContent: spaceBetween` 分三大区域：header / middle / button，避免 itemMargin 累加超高。
- header Row：height 20，icon 18 + gap 6 + 标题文本，alignItems center。
- middle Column：itemMargin 4，meetingName(18fp,~22) + timeLabel(14fp,~18) + reminder(10fp,~14)，组内高 ≈ 22+18+14+8 = 62。
- button：width 116、height 34、borderRadius 17，贴底；底边距由 spaceBetween 自动落到安全区底部（≈12vp 内）。
纵向：20 + 62 + 34 = 116，正好填满内容区，spaceBetween 吸收余量，无溢出。
横向：所有受保护文本与按钮宽度 116 = 内容区宽度，无截断。

## 颜色来源
- root.bg = background_primary(light) #ffffffff
- title = font_secondary #99000000
- meetingName = font_primary #e5000000（主数值/主答案）
- timeLabel = font_emphasize / brand #ff0a59f7（强调时间）
- reminder = font_tertiary #66000000（弱提示）
- button.bg = brand / comp_background_emphasize #ff0a59f7（强动作=进入专注模式）
- button.label = font_on_primary #ffffffff
单一主场景色族 = brand 蓝；无状态色滥用，无渐变。

## 事件
clickToDeeplink → 设置 com.huawei.hmos.settings / MainAbility，uri=intelligent_scene_entry
（manifest 中「打开系统设置中的情景模式，用户可以打开免打扰或专注模式」），意图匹配。
点击行为只写 DSL onClick，不写入 CardSpec。

## 数据
静态卡片：CardSpec 只含 suggestSize=2x2，无 dataBindings/refreshPlan。
展示值用完整表达式 `{{ $__dataModel.xxx }}` 读取 DataModel，已在 updateDataModel 初始化。

## 校验
- L0 协议：三行 JSONL、v0.9、extended.catalog、仅用 Form 组件、root shell 完整 ✓
- L1 布局：内容区预算成立、受保护文本完整、按钮热区 116x34≥24vp、贴底 ✓
- L2 内容：一个主答案 + 2 支撑事实 + 1 动作，职责互斥，无事实重复 ✓
- 颜色：全部可回溯官方 token，单主色族 ✓
