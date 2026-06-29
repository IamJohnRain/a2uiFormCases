# case-2 耳机音乐卡片 — 布局理由与校验

## 需求
刚连上耳机，展示：耳机名称、左/右耳电量、每日推荐音乐、我的心动歌单。

## 尺寸决策
- 4 个信息组（耳机名 / 左右电量 / 每日推荐 / 心动歌单）超出 2x2 的可读容量，选 **2x4**（300x140，padding 12，内容区 276x116）。
- 采用 split 左右分区：左列（设备+电量）138vp + 竖分割线 + 右列（两个音乐背板）109vp。

## 横向预算
- root padding 12 -> 内容 276。138(左) + 12(itemMargin) + 1(分割线) + 12(itemMargin) + 109(右) = 272 <= 276 ✓
- 左列电量行：标签16 + 6 + 进度梖74 + 6 + 百分比36 = 138 ✓
- 右列背板 padding 8 -> 内容 93；图标14 + 4 + 标題75 = 93 ✓

## 纵向预算
- 左列 height 116：名称行24 + 10 + 电量18 + 10 + 电量18 = 80，center 对齐。
- 右列 height 116：背板50 + 8 + 背板50 = 108 <= 116 ✓

## 信息职责（互斥）
- 耳机名=对象；左/右电量=两个独立实际值（进度条可视化 + 百分比唯一主数值，不重复）；每日推荐=推荐曲目；心动歌单=收藏入口。

## 颜色来源
- 场景拓展色 music.ambient：baseSources = multi_color_06 (#ac49f5)。
- root 线性渐变 #ff7a2fd6 -> #ffac49f5（同一紫色族低/中明度，rootStart/rootEnd）。
- 前景用 on-color：主文字 #ffffffff，次要 #b3ffffff；背板 #1fffffff，分割线 #33ffffff，进度槽 #40ffffff。均为白色 alpha 叠加，无第二个高饱和前景。

## 素材
- 耳机 icon_earphone.png、音乐 icon_music.png、心形 icon_like.png，均来自素材库“戴耳机播控”场景，显式 width/height + objectFit contain。

## 事件
- 事件 manifest 无音乐/设置以外可用跳转目标，不伪造 onClick，静态展示。

## 数据能力
- data-capability 目录仅有天气/日历，无耳机电量/音乐能力，故为静态卡片，cardspec 只写 suggestSize，不虚构 dataBindings/refreshPlan。

## 校验
- genui 三行 JSONL 均 json.loads 通过；cardspec JSON 通过。
- version v0.9 / catalog extended / 2x4 / 300x140 / borderRadius 22 / clip true 一致。
