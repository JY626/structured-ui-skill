---
name: structured-ui
description: >-
  Cross-platform UI design system and physics micro-animation guide inspired by the
  award-winning Structured – Daily Planner app for iOS, Android, and Web.
  Use when designing, refactoring, or auditing a consumer app, planner, tracker, or
  productivity tool that needs a warm timeline layout, rounded typography, cohesive
  multi-theme palettes with isolated semantic icons, floating capsule navigation,
  and tactile spring micro-interactions.
---

# Structured UI — Cross-Platform Design & Motion Skill

本 Skill 将获奖应用 **Structured – Daily Planner** 的视觉哲学、全站级主题边界规则与物理微动效体系沉淀为 **iOS / Android / Web 三端通用** 的工程指南，帮助智能体输出克制、温润、高触感且不堆砌边框的现代产品界面。

---

## 何时启用

- 用户要求为 **iOS (SwiftUI)**、**Android (Jetpack Compose / Material)** 或 **Web (React / Vue / CSS / Tailwind)** 新建或重构 UI 与动效。
- 涉及日程、任务流、健身打卡、习惯追踪、时间轴、数据看板或移动端风格的桌面/网页应用。
- 需要设计既支持多套主题色实时切换、又能在单色/黑白主题下保持语义清晰与高对比度的设计系统。

---

## 六大核心设计法则

### 1. 0.5 秒视觉脊柱（Visual Spine）
- **先定形状再写代码**：用户打开页面 0.5 秒内必须看清“主阅读动线”与“唯一主动作”。
- **连续时间轴**：用左侧细轨道线（`timelineRail`, 2px/2pt）串联圆角 Squircle 图标胶囊（`40–44px`）与内容卡片，使节奏与进度空间化。
- **顶部速览 + 底部悬浮 `+`**：顶部放紧凑的 7 日横条或焦点进度环卡片；唯一高频创建入口使用右下角悬浮圆形 `+` 按钮（`56×56px`），避免厚重的底部全宽占位条。

### 2. 暖纸感浅色 & 温润炭灰深色（Surface Hierarchy over Heavy Borders）
- **拒绝 1px 硬灰框套娃**：不要用明显灰边框套大卡、大卡里再套带框小输入框。
- **靠三层表面区分层级**：
  - `canvas`（画布底色）：浅色带微暖纸感（如 `#FAF5F3`），深色用低疲劳炭灰（如 `#161213` / `#121215`），避免刺眼纯白与冷硬死黑死循环。
  - `surface1`（主卡片）：柔和圆角（`20–22px`），浅色靠轻柔扩散投影分层，深色靠极微弱内反光边（`white @ 6%`）分层。
  - `surface2`（次级输入槽 / 胶囊）：嵌入主卡的柔雾底块，无需外框。

### 3. 主题渗透深度 vs. 语义色隔离边界（Critical Rule）
设计多主题切换时，必须严格区分 **“主题表现层”** 与 **“固定语义层”**：

| 元素类别 | 是否随主题切换 | 设计规范 |
| :--- | :---: | :--- |
| 画布 `canvas` / 卡片 `surface1` / 输入槽 `surface2` | **是** | 随主题冷暖微调（如蓝系偏雾蓝、黑白系完全中性无偏色） |
| 进度条、容量横条、图表柱状、日历打卡圆、主 CTA、Tab 高亮 | **是** | 紧跟当前主题主色与柔和底色（`accentFill` / `accentSoftBg`） |
| 分类阶梯色（如任务类型 / 肌群 / 标签） | **是** | 映射到当前主题的同色系阶梯，**极简黑白主题下必须 100% 保持灰阶，禁止残留杂色** |
| **固定语义图标**（连续火焰、打卡完成绿、荣誉奖杯金、警示/热身琥珀、健康同步绿、危险删除红） | **否** | 保持各自固有语义色，不随主题换色而被强行同化，确保一眼识别状态 |
| 实心强调色上的文字 / 图标（`onAccent`） | **是（动态反色）** | 根据背景明度切换黑/白字（如暗黑模式极简黑白为白圆配黑字 `#141416`） |

### 4. 圆润精密排版（Rounded Display + Tabular Numbers）
- **字体组合**：核心标题、数值、日历格与胶囊标签使用圆润无衬线字体（iOS `SF Pro Rounded`，Android 自定义圆润家族或几何 Sans + `fontFeatureSettings = "tnum"`，Web `ui-rounded` / `Plus Jakarta Sans` + `font-variant-numeric: tabular-nums`）。
- **数字主从**：关键数值放大加粗，单位（`kg`、`min`、`组`、`%`）缩小字号并降为次级灰且基线对齐。
- **文案克制**：禁止使用中点 `·` 作为信息分隔符（改用间距或换行）；不写冗余解释性废话；不强行造词。

### 5. 物理弹簧微交互与触觉反馈（Tactile Spring Motion）
动画服务于状态因果关系，不搞炫技飞入：
1. **按压微缩**：可点击卡片与按钮按下轻缩至 `0.94–0.96`，松手弹性回弹。
2. **涟漪勾选环（Signature Ripple Check）**：圆框勾选完成时，实心填充缩放弹出 + 圆心向外扩散消散一圈柔和光环 + 成功触觉反馈。
3. **滚轮数字翻滚**：数值增减或切换统计周期时触发原生数字滚转过渡（`.numericText` / `AnimatedContent`）。
4. **滑动高亮胶囊**：分段控制器与底部 Dock 选中态用共享几何元素（`matchedGeometryEffect` / `layoutId` / `animateDpAsState`）丝滑滑移。
5. **悬浮底栏与防遮挡**：切 Tab 采用 250ms ease-out 淡入轻缩放并保留滚动位置；**进入二级详情页时底部悬浮导航必须干净隐去**；滚动容器底部必须留足安全间距（约 `92px` + Safe Area），绝不遮挡最底部的按钮。
6. **无障碍降级**：全程尊重系统减弱动态效果（Reduce Motion）。

### 6. 小屏优先与安全热区
- 以移动端紧凑视口（`375×667pt` / `360×640dp`）作为基准校验，不换行截断、不溢出屏幕。
- 所有主要点击热区不小于 `44×44pt` / `48×48dp`。

---

## 渐进披露参考指南（按需加载）

请先查看基础令牌与动效规范，再根据当前项目技术栈读取对应平台文档：

1. **全站色彩、语义隔离与排版令牌**：阅读 [design-tokens.md](./references/design-tokens.md)（机器可读 JSON 见 [tokens.json](../../examples/tokens.json)）
2. **物理动效参数与微交互详规**：阅读 [motion-and-haptics.md](./references/motion-and-haptics.md)
3. **按目标平台读取落地代码范式与防坑清单**：
   - **iOS (SwiftUI iOS 17+)**：[platform-ios.md](./references/platform-ios.md)
   - **Android (Jetpack Compose / Material 3)**：[platform-android.md](./references/platform-android.md)
   - **Web (React / Framer Motion / CSS Variables)**：[platform-web.md](./references/platform-web.md)

---

## AI 执行流程

1. **盘点页面与状态边界**：先梳理目标应用现有页面、导航层级与核心动作，明确哪些元素属于“主题层”，哪些属于“独立语义层”。
2. **建立集中式 Theme Token 层**：按 `design-tokens.md` 实现可动态观测的 Theme Manager / Provider，将画布、卡片、输入槽、分类阶梯与独立语义色一次收敛，禁止在组件内硬编码随机十六进制颜色。
3. **逐页重构视觉骨架与微交互**：优先落地顶部速览、时间轴流、右下角悬浮 `+` 按钮、涟漪勾选框、数字滚转与滑动胶囊底栏，并保证所有滚动页面底部预留充足安全空间。
4. **双模式 & 主题切换自检**：分别在**浅色 / 深色**以及**极简黑白 / 彩色主题**下编译或预览检查：
   - 切换为极简黑白时，检查首页/列表/横条/分类图标是否完全无彩色残留；
   - 切换任意主题时，检查连续火焰/打卡绿/奖杯金/警告红是否依然保持独立语义清晰；
   - 进入二级详情页时，检查底部悬浮导航栏是否自动隐去、最底部操作按钮是否完全不被遮挡；
   - 确认开启系统 Reduce Motion 时动效安全退化，编译零警告。
