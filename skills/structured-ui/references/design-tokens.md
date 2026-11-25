# Design Tokens & 色彩边界规范

Structured 风格的核心在于**温和的纸感对比、克制的色彩比例，以及严格的“主题色 vs 语义色”边界**。

---

## 1. 表面层级体系（Surface Hierarchy）

每一套主题在浅色（Light）与深色（Dark）下均提供 4 个基础层级，**禁止使用硬灰 `1px` 边框分隔卡片与内部输入槽**：

| 层级 Token | 用途 | 浅色模式特点 | 深色模式特点 |
| :--- | :--- | :--- | :--- |
| `canvas` | 页面底层背景 | 微暖/微雾纸感（非纯白 `#FFFFFF`） | 温润低疲劳深灰（非冷黑 `#000000`） |
| `surface1` | 主卡片、时间轴任务块 | 纯白 `#FFFFFF` + 轻扩散暖阴影 `0 4px 12px` | 较画布亮 1 阶的炭灰 + `0.5px` 内反光边（`white @ 6%`） |
| `surface2` | 嵌入卡片的输入槽、未选中胶囊、次级按钮 | 柔雾灰底（随主题冷暖微调） | 比 `surface1` 更亮一层的深灰表面 |
| `timelineRail` | 时间轴左侧垂直连接轨 | 低透明暖灰/冷灰（11%） | `white @ 11%` |

---

## 2. 5 套全站级主题色板

全部 Hex 已校准明暗对比度，完整机器可读 JSON 见 [tokens.json](../../../examples/tokens.json)：

1. **Coral Rose / 柔珊瑚（默认主推）**
   - Light：`canvas #FAF5F3` · `surface1 #FFFFFF` · `surface2 #F3EAE7` · `accentFill #DE6B62` · `accentTint #C8544B`
   - Dark：`canvas #161213` · `surface1 #221C1E` · `surface2 #2F2729` · `accentFill #EA847B` · `accentTint #F2948C`
2. **Warm Peach / 暖杏橙**
   - Light：`canvas #FAF6F0` · `surface1 #FFFFFF` · `surface2 #F3ECE2` · `accentFill #DC7A3B` · `accentTint #C26224`
   - Dark：`canvas #161310` · `surface1 #221E1A` · `surface2 #2F2923` · `accentFill #F29B63` · `accentTint #F5A875`
3. **Sky Mist / 晴空蓝**
   - Light：`canvas #F3F7FB` · `surface1 #FFFFFF` · `surface2 #E7EFF7` · `accentFill #4A8AD4` · `accentTint #3674BD`
   - Dark：`canvas #101318` · `surface1 #191F27` · `surface2 #232C37` · `accentFill #6FA8EB` · `accentTint #82B6F2`
4. **Sage Mint / 鼠尾草**
   - Light：`canvas #F3F8F5` · `surface1 #FFFFFF` · `surface2 #E6F1EC` · `accentFill #469E77` · `accentTint #348260`
   - Dark：`canvas #101614` · `surface1 #18211E` · `surface2 #222F2A` · `accentFill #6EC29C` · `accentTint #80CFA9`
5. **Monochrome / 极简黑白**
   - Light：`canvas #F4F4F6` · `surface1 #FFFFFF` · `surface2 #EAEAEF` · `accentFill #18181B` · `onAccent #FFFFFF`
   - Dark：`canvas #0C0C0E` · `surface1 #18181B` · `surface2 #252529` · `accentFill #F5F5F7` · `onAccent #141416`

> [!IMPORTANT]
> **反色文字规则（`onAccent`）**：在彩色主题下实心填充配纯白 `#FFFFFF`；但在 **极简黑白 Dark 模式** 下，实心填充是浅白 `#F5F5F7`，因此 `onAccent` **必须切换为深墨 `#141416`**，否则按钮文字和日历圆内数字会直接隐身。

---

## 3. 分类阶梯色 & 主题隔离矩阵

当页面存在多种业务分类（如 4 个训练日类型、4 个日程大类）时，**不要把“红色/蓝色/绿色/橙色”写死到业务枚举里**：

- **正确做法**：由当前主题提供 4 档同色系/邻近色阶梯 `categoryScale[0...3]`（例如晴空蓝下全部在蓝系明度区间内区隔；**极简黑白下 4 个分类全部取中性灰阶**）。
- **为什么这样设计**：用户切换「极简黑白」时，不会出现“页面其他地方黑白，结果列表进度横条和图标还是橙色/绿色”的割裂感。

---

## 4. 固定语义色（不受主题影响）

以下颜色携带明确的直觉心理暗示，**固定为全局常量，不随主题切换而变色**：

| 语义 Token | Light | Dark | 适用场景 |
| :--- | :--- | :--- | :--- |
| `streakFlame` | `#E56032` | `#FF8A5B` | 连续活跃天数火焰图标 |
| `checkinSuccess` | `#329E6A` | `#63CA98` | 打卡完成绿标、健康同步状态点 |
| `trophyGold` | `#CE8E16` | `#F4BE4F` | 最长连续记录奖杯、里程碑勋章 |
| `warningAmber` | `#B36818` | `#F7C17C` | 热身组、暂停态、轻提醒 |
| `destructive` | `#DC4C46` | `#F2736D` | 删除、撤销、不可逆危险操作 |

---

## 5. 排版、圆角与间距基准

- **圆角体系（Concentric Radii）**：
  - 主卡片 `cardRadius = 22px`
  - 内部控件/输入槽 `controlRadius = 14px`（外圆角 `22` ≈ 内圆角 `14` + 内边距 `8–16`，保持视觉同心）
  - 时间轴图标 Squircle `14px`
  - 悬浮 Dock / 按钮 `9999px`（全胶囊 / 正圆）
- **分割线**：统一 `0.5px`（或 `1px` 低透明度 `8%`），禁止重线条。
- **字号层级（克制在 3–4 档内）**：
  - 核心指标大数字：`24–32px` Bold Rounded + 等宽数字
  - 卡片标题：`17px` Bold Rounded
  - 正文 / 输入值：`15–16px` Medium/Semibold
  - 辅助说明 / 标签 / 单位：`12–13px` Medium（最小不低于 `12px`）
- **文案规则**：绝对禁用中点 `·` 作为分隔符，多个辅助字段之间用 `8–10px` 水平留白区隔。
