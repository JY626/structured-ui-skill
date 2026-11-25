# 物理动效与触觉反馈规范

Structured 的交互灵魂在于**“物理真实感与微剂量愉悦（Micro-Delight）”**：每一个弹簧与涟漪都对应明确的用户输入，不做无意义的花哨飞入。

---

## 1. 四档核心动效曲线

| 预设名称 | iOS (SwiftUI) | Android (Compose) | Web (Framer / CSS) | 适用场景 |
| :--- | :--- | :--- | :--- | :--- |
| `snappy` | `spring(response: 0.30, dampingFraction: 0.78)` | `spring(dampingRatio = 0.78f, stiffness = 450f)` | `type: "spring", stiffness: 420, damping: 32` | 按钮微缩、滑动胶囊、展开/收起面板、增减数字 |
| `bouncy` | `spring(response: 0.36, dampingFraction: 0.66)` | `spring(dampingRatio = 0.65f, stiffness = 360f)` | `type: "spring", stiffness: 360, damping: 22` | 圆形勾选框完成弹出、选中卡片图标弹跳 |
| `gentle` | `spring(response: 0.44, dampingFraction: 0.84)` | `spring(dampingRatio = 0.84f, stiffness = 260f)` | `type: "spring", stiffness: 260, damping: 28` | 进度条/柱状图生长、切换统计周期过渡 |
| `tabSwitch` | `easeOut(duration: 0.25)` | `tween(250, easing = FastOutSlowInEasing)` | `duration: 0.25, ease: [0.22, 1, 0.36, 1]` | 底部主导航栏切页淡入与轻缩放 |

---

## 2. 五大招牌微交互详规

### 2.1 弹性按压微缩（Press Micro-Scale）
- **视觉表现**：手指/鼠标按下时，目标控件在 `snappy` 曲线下平滑缩至 `0.94–0.96`（小型圆形按钮可用 `0.88–0.91`），透明度微降至 `0.92`；松开后自然回弹。
- **注意**：对 `List` 整行导航入口谨慎使用过强缩放（`0.97` 为宜），避免与系统滑动删除手势打架。

### 2.2 圆形涟漪勾选框（Signature Ripple Check Circle）
完成某一组训练或一条待办时的标志性仪式感，由 **4 个叠层动作** 同步触发：
1. **未完成态**：空心圆环（`strokeWidth: 1.75–2px`，低饱和灰描边）。
2. **实心填充弹出**：状态变为完成时，实心主题色圆以 `bouncy` 弹簧从 `scale 0.65 -> 1.0` 弹出，中心对勾图标执行一次轻微弹跳。
3. **涟漪光圈扩散（Ripple Ring）**：在圆环背后同心叠加一个与主色同色的细圆环，在 `0.42s ease-out` 内从 `scale 0.9 -> 1.65`、`opacity 0.55 -> 0` 向外散开消隐。
4. **整行柔和唤醒 + 触觉**：整行卡片背景平滑渗入当前主题 `accentSoftBg`，同时触发一次 **Success 级触觉反馈**。

### 2.3 滚轮数字翻滚（Numeric Ticker）
- 适用于重量、次数、完成进度、倒计时与统计 KPI 数字。
- 数值发生变化时，仅变化的那几位数字做上下微幅滚转过渡，**务必开启等宽数字（tabular/monospaced digits）**，防止数字跳动导致整行左右抖动。

### 2.4 滑动高亮胶囊（Sliding Pill Indicator）
- 适用于分段控制器（如周/月/全期、浅色/深色）和底部悬浮导航栏（Floating Dock）。
- 选中背景胶囊使用共享几何标识（SwiftUI `matchedGeometryEffect`、Compose `animateDpAsState`、Framer Motion `layoutId`）在选项间水平滑移，而非原地生硬闪烁切换。

### 2.5 悬浮磨砂底栏（Floating Frosted Dock）的切页与自动收起
- **切页不丢状态**：主 Tab 容器保持各 Tab 实例存活（隐藏态用 `opacity: 0` + `scale: 0.985` + 禁用点击），切换时用 `250ms ease-out` 淡入回 `1.0`。
- **二级页自动隐去（防遮挡核心）**：一旦从主列表 Push 进入详情页或二级设置页，悬浮底部导航栏必须平滑向下滑出（`translateY + opacity`）；返回根页时弹性滑回。
- **底部滚动留白**：所有存在悬浮 Dock 或悬浮 `+` 按钮的滚动页面，底部滚动内边距必须 ≥ Dock 高度 + 底部外边距 + 安全区（移动设备上通常为 `88–96px`；若有右上/右下角悬浮 FAB，额外增加 `56px` 错位空间）。

---

## 3. 触觉反馈（Haptics）分级表

| 交互场景 | iOS 反馈 | Android 反馈 | Web |
| :--- | :--- | :--- | :--- |
| 完成勾选 / 达成目标 | `.sensoryFeedback(.success, ...)` | `HapticFeedbackType.LongPress` 或 `CONFIRM` (API 30+) | — |
| 切换 Tab / 分段 / 主题 / 日历点选 | `UISelectionFeedbackGenerator().selectionChanged()` | `TextHandleMove` / `CLOCK_TICK` | — |
| 点击 `+` 新建 / 调节数值（+2.5 / +1） | `UIImpactFeedbackGenerator(style: .light/.medium)` | `TextHandleMove` | — |

---

## 4. 无障碍降级（Reduce Motion）

必须监听系统的减弱动态效果设置：
- iOS：`@Environment(\.accessibilityReduceMotion) private var reduceMotion`
- Android：检查 `Settings.Global.ANIMATOR_DURATION_SCALE == 0f` 或 `LocalInspectionMode`
- Web：`@media (prefers-reduced-motion: reduce)` / Framer Motion `<MotionConfig reducedMotion="user">`

开启 Reduce Motion 时：跳过弹性缩放、涟漪扩散与位移动画，保留清晰的状态色彩切换或极短线性透明度过渡。
