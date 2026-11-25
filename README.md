# structured-ui

一套受 Apple Design Award 提名 / App Store Award 获奖应用 **Structured – Daily Planner** 启发的**跨平台 UI 设计系统与物理微动效 Agent Skill**，同时适用于 **iOS (SwiftUI)**、**Android (Jetpack Compose)** 与 **Web (React / Framer Motion / CSS)**。

[English](./skills/structured-ui/SKILL.md) | 简体中文

---

## 核心设计理念

1. **0.5 秒时间轴秩序（Visual Timeline Spine）**：用左侧连续垂直脊柱 + 圆润 Squircle 图标胶囊串联日程、任务或记录块，配合右下角悬浮圆形 `+` 主动作按钮，扫视即可把握节奏与优先级。
2. **暖纸感浅色 & 温润炭灰深色**：放弃刺眼纯白与冷硬灰边框。浅色采用暖象牙纸感画布搭配轻暖投影；深色采用温润深炭黑与极微弱内反光边，长时间使用不疲劳。
3. **主题深度渗透 vs. 语义色隔离边界**：
   - **随主题联动**：画布底色、卡片层级、次级输入槽、进度条/横条、进度环、图表、时间轴分类阶梯与悬浮胶囊底栏跟随主题色整体切换（极简黑白模式下做到 100% 纯粹灰阶，无彩色泄漏）。
   - **语义色独立**：连续火焰、打卡完成绿、荣誉奖杯金、警示/热身琥珀与危险红保持独立固定语义，不被主题色强制同化；实心主色按钮/圆点上的反色文字动态计算对比度。
4. **圆润精密排版**：标题与核心数值优先采用 Rounded 字体搭配等宽数字（Tabular / Monospaced Digits），支持数值滚轮过渡；小屏优先，禁用中点 `·` 作为 UI 分隔符。
5. **物理弹簧微交互与触觉反馈**：包含按压微缩、圆形涟漪勾选框（Ripple Check Circle）、滚轮数字翻滚、滑动高亮胶囊与自动隐显悬浮底栏，并完整尊重系统的“减弱动态效果（Reduce Motion）”。

---

## 目录结构

```text
structured-ui-skill/
├── README.md
├── LICENSE
├── examples/
│   └── tokens.json                         # 跨平台 W3C Design Tokens 示例（5 套主题 + 语义色 + 圆角/动效）
└── skills/
    └── structured-ui/
        ├── SKILL.md                        # Skill 主入口（触发条件、6 大设计法则、渐进披露路由、验收清单）
        └── references/
            ├── design-tokens.md            # 完整 5 套全站级主题色板、语义隔离矩阵与排版规则
            ├── motion-and-haptics.md       # 4 档物理动效曲线、5 大微交互规范与减弱动态效果降级
            ├── platform-ios.md             # SwiftUI (iOS 17+) 组件范式与防坑指南
            ├── platform-android.md         # Jetpack Compose / Material 3 组件范式与防坑指南
            └── platform-web.md             # CSS Variables + React / Framer Motion 落地指南
```

---

## 快速安装

### 1. Antigravity / Claude Code / Codex（`npx skills`）

```bash
npx skills add https://github.com/JY626/structured-ui-skill
```

### 2. 手动安装到全局 Skills 目录

- **Antigravity**：
  ```bash
  git clone https://github.com/JY626/structured-ui-skill.git ~/.gemini/config/structured-ui-skill
  ln -sfn ~/.gemini/config/structured-ui-skill/skills/structured-ui ~/.gemini/config/skills/structured-ui
  ```
- **Claude Code**：
  ```bash
  git clone https://github.com/JY626/structured-ui-skill.git ~/.claude/structured-ui-skill
  ln -sfn ~/.claude/structured-ui-skill/skills/structured-ui ~/.claude/skills/structured-ui
  ```
- **项目级共享（团队协作）**：将 `skills/structured-ui` 复制到项目根目录 `.agents/skills/structured-ui` 并提交到版本库。

---

## License

MIT © [JY626](https://github.com/JY626)
