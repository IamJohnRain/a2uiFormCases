# Case 2 Notes

- 布局：采用 2x4，左侧 176vp 毛玻璃主面板展示深圳、31℃、天气；右侧 90vp 承载高温预警、感冒指数和联系父母按钮，避免温度、预警和 CTA 在 2x2 内拥挤。
- 数据：可见动态字段优先使用 `{{ $__dataModel... }}`；天气能力写入 `/data/weather`，关怀字段保留在 `/data/care`，包含高温预警、感冒指数“易发”和父母电话 `15012345678`。
- 事件：电话按钮使用已声明 `clickToCallPhone` capability，参数来自 DataModel；CardSpec 不写点击行为。
- 颜色：root 渐变基于 `multi_color_08` 和 `multi_color_02` 的深蓝到天空蓝天气场景；半透明白色面板与白色 CTA 模拟毛玻璃层次，预警文字使用 `warning` token 映射色。
- 自检：JSONL 为三行，组件限定在 Form 支持集；root 与 surface 均为 300x140、圆角 22、clip true；按钮 90x34 满足点击热区，主要文本均设定固定宽高且不使用省略。
