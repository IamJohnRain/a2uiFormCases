# 抖音防沉迷卡片 case-1 设计笔记

## 场景与信息取舍
- 主答案：今日已观看时长（`45分钟`，32fp 主数值）。
- 支撑事实：① 每日限额（header 右侧 `限60分钟`，10fp 弱提示）；② 进度条（已用/限额可视化）+ 剩余时长文案。
- 主动作：`开启专注` 按钮，跳转系统设置情景模式（专注/免打扰），合法 supportedTarget。
- 事实等价类去重：进度条只做可视化，旁边只写"剩余 15分钟"（唯一派生主事实），不复述 45/60 两端值。

## 尺寸与布局
- 尺寸 2x2（140x140，root padding 12 → 内容区 116x116，borderRadius 18，clip）。
- 原型：top-hero-pill 变体（标题行 → hero 主数值 → 进度组 → 底部动作胶囊）。
- root Column itemMargin 8，spaceBetween。
- 纵向预算：header 18 + 主数值 40 + 进度组 24 + 按钮 30 = 112，+ 3*itemMargin(8)=24 → 与 padding 共占，spaceBetween 吸收余量，未溢出 116。
- 横向：所有主元素 width 116，等于内容区宽，左右边界共享。
- header Row spaceBetween：标题 12fp(4字≈48) + 限额标签 10fp(`限60分钟`≈55) + gap，总和 < 116，成立。
- 按钮 height 30 > 24vp 热区，width 116 与内容同宽，贴底（底边距由 spaceBetween + padding 控制在安全范围）。

## 颜色来源
- root 背景 `#FFFFFFFF` = background_primary(light)。
- 标题 `#E5000000` = font_primary；主数值同 font_primary。
- 限额标签 `#66000000` = font_tertiary（弱提示）；剩余 `#99000000` = font_secondary。
- 进度条 + CTA 填充 `#FFE64566` = multi_color_07(light) 作单一主场景色族（提醒/克制类红，服务防沉迷语义）；CTA 前景 `#FFFFFFFF` = font_on_primary。
- 仅一个主场景色族，未叠加第二高饱和前景。未误用 warning/alert 作整卡主题。

## 数据契约
- 静态展示卡：CardSpec 只有 suggestSize=2x2，无 dataBindings/refreshPlan（无对应防沉迷 data-capability，按降级原则做静态卡）。
- DataModel 初始化 usage 节点：usedText/usedMinutes/limitText/limitMinutes/remainText，供表达式与 Progress 读取。
- 表达式优先：content 用 `{{ ... }}`；Progress value/total 用 `{{ ... }}`。

## 校验
- 模型内置校验通过：L0 协议（三行 JSONL、catalog、尺寸一致）、绑定（所有引用在 DataModel 中）、布局预算、颜色来源、事件能力(clickToDeeplink + 合法 target)、信息职责互斥。
