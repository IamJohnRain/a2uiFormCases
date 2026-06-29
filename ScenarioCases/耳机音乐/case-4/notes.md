# case-4 耳机桌面小组件 — 布局理由与校验日志

## 需求归纳
- 主对象：耳机 Free Clip 2（连接状态 + 续航一眼可读）
- 支撑事实：左耳 47%、右耳 95% 电量
- 主动作：两个快捷入口「每日30首」「心动歌单」
- 视觉：浅灰、磨砂玻璃、柔和光影、干净高级、带耳机识别元素

## 尺寸取舍：2x4
2x2 上限为 1 个显式动作，本卡有两个并列快捷入口（两个动作区）+ 设备状态组，
属于「两个并列动作」「核心内容需并列理解」的具体失败点，按 layout-system 升级 2x4。
采用 split-action 原型：左 168vp 状态面板 + 10vp 间距 + 88vp 动作列 = 266 ≤ 276。

## 横向预算（root padding 12 → 内容 276×116）
- 168 + 10(itemMargin) + 88 = 266 ≤ 276 ✓（留 6vp 余量）

## 状态面板（168 宽，padding 12 → 内宽 144；高 116 → 内高 92）
- title_row：icon20 + gap6 + text118 = 144 ✓
  - "Free Clip 2" 11 个英文/数字字符 ≈ 11×0.6×16 ≈ 106 ≤ 118，完整显示，无省略
- battery 行：lbl16 + gap6 + bar78 + gap6 + pct36 = 142 ≤ 144 ✓
- 纵向：title≈20 + 8 + 18 + 8 + 18 = 72 ≤ 92，justifyContent center ✓

## 动作列（88 宽，116 高）
- 两按钮 48 + 48 + gap8 = 104 ≤ 116，居中 ✓
- 按钮 88×48，远大于 24vp 热区；与左面板共享垂直边界

## 信息职责（互斥，无等价类重复）
- 对象 = title_text；状态/续航 = 左右两条 battery 行（label 仅区分左右声道，非重复事实）
- 进度条只作可视化，旁边百分比是该声道唯一主数值，不复述
- 动作入口 = 两个 Button，各自语义不同

## 颜色来源
- root 渐变：scene=music.ambient，baseSources=background_secondary(#fff1f3f5)/background_tertiary，
  派生低饱和浅灰玻璃表面 #FFF5F6F8 → #FFEAECEF（线性，135°，2 stop）
- 面板/按钮：白色磨砂层 #B3FFFFFF / #CCFFFFFF（中性表面，低 alpha 白，玻璃感），
  边框 #33FFFFFF / #1A000000 作轻分隔
- 文本：font_primary #E5000000 / font_secondary #99000000
- 电量条：multi_color_04 健康绿 #FF64BB5C（真实续航状态），轨道 #1A000000（低 alpha 黑）
- 单一主场景色族（中性灰玻璃）+ 一个状态色（绿），符合克制要求

## 资产
- icon_earphone.png（戴耳机场景，承载耳机识别）以 20vp 小图标置于标题前；
  放弃大面积背景耳机图：catalog 无 opacity 样式，全不透明大图会压过磨砂层与文本，
  改由标题图标 + 玻璃渐变达成「干净高级 + 耳机识别」。

## 校验
- L0：三行 JSONL，version v0.9，extended catalog，2x4=300×140，root borderRadius22/clip ✓
- 仅用 Row/Column/Image/Text/Progress/Button，children 全为 id 数组 ✓
- onClick 使用已声明 clickToDeeplink + supportedTargets 合法组合 ✓
- 展示值用完整表达式 {{ }}，DataModel 路径 device.name / battery.left / battery.right 均可推导 ✓
- 静态卡 cardspec 只写 suggestSize，未虚构 dataBindings/refreshPlan ✓

## 备注
- 两个快捷入口当前复用同一可达 deeplink 目标（manifest 未声明音乐 App 目标）；
  端侧接入真实音乐 App bundleName/abilityName 后替换即可，DSL 结构不变。
