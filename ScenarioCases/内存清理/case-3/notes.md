# case-3 手机状态/内存清理卡片 笔记

## 需求拆解
用户：手机卡，想看内存用了多少 / 剩多少空间 / 电量，并能"实时看到手机状态、释放内存加速"。
- 主答案：内存占用（卡顿主因）→ 环形进度作 hero。
- 支撑事实：剩余存储空间、电量。
- 主动作：一键加速。

## 尺寸选择：2x4
2x2 上限为 1 hero + 1 支撑组 + 1 动作。本需求需并列阅读 3 个指标（内存/存储/电量）+ 1 动作，
2x2 放不下且会信息打架，按 layout-system 的 split-action 升级 2x4。
- 内容区 276x116，root padding 12。
- 左列 164（内存环 hero + 标签），itemMargin 12，右列 88（存储/电量/动作）。
  164 + 12 + 88 = 264 ≤ 276，横向预算成立。
- 右列纵向：storage_row ~28 + battery_row ~28 + pill 30 + 2*10 = 126? 重核：
  实际文本两行(10fp+16fp)约 12+18=30，两组共 60，pill 30，itemMargin 10*2=20 → 110 ≤ 116，成立。

## 颜色来源
- root 背景 comp_background_primary(light) = #FFFFFFFF。
- 主色族单一：brand / multi_color_08 = #FF0A59F7，用于内存环和加速胶囊（动作/清理语义 action-fill）。
- 文本 font_primary #E5000000 / font_secondary #99000000；caption font_tertiary 同族。
- 加速胶囊为饱和品牌底，前景用 font_on_primary #FFFFFFFF。
- 未使用 warning/alert/confirm，避免把状态色当主题；电量为中性展示而非告警。

## 信息职责互斥
- 内存：环（比例可视化）+ 中心 %（主数值）+ 标签 已用/总量（绝对值），比例与绝对值是不同事实，不算重复。
- 存储：剩余空间 GB（一个事实）。
- 电量：百分比（一个事实）。
- 动作：一键加速。

## 动作处理（重要）
event-capability manifest 当前仅支持 clickToDeeplink→设置/情景模式、天气、闹钟，及 clickToIntent→日历。
没有"内存清理/手机管家加速"的合法 deeplink/intent 目标。按 click-event.md "不要伪造点击能力"，
故"一键加速"以视觉 CTA 胶囊（Text 样式）呈现，未挂 onClick，避免伪造跳转。
如端侧补充清理/手机管家 event-capability manifest，再将该胶囊替换为 Button 并接入合法 call。

## 动态数据 / 实时刷新
"实时看到手机状态"需端侧设备状态能力（内存/存储/电量），但 data-capability/ 目录未声明该能力，
按 cardspec.md "不要编造能力"，本卡输出为静态降级：cardspec 只含 suggestSize，
DataModel 初始化 /data/status 根结构与示例值（非真实隐私）。
端侧具备设备状态 data-capability 后，可补 dataBindings(writeResultTo:/data/status) + refreshPlan(onVisible/periodic) 实现实时刷新。

## 校验
- L0：三行 JSONL，version v0.9，catalog extended，2x4 尺寸一致（300x140 / borderRadius22 / clip）。
- L1：横纵预算均成立，受保护文本（%、GB、CTA）一行完整，无 ellipsis。
- 绑定：所有表达式路径在 updateDataModel.value 可找到。
- 颜色：均可回溯 token，单一主色族。
