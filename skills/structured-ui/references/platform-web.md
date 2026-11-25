# Web (CSS Variables + React / Framer Motion) 落地指南与防坑清单

本指南提供面向现代 Web 应用（React / Next.js / Vite + Framer Motion 或纯 CSS）的落地模式，兼顾桌面与移动端 PWA 体验。

---

## 1. 全站级 CSS Variables 主题系统

通过根节点 `data-theme` 与 `data-mode` 属性集中控制全站颜色变量（包括画布、卡片、输入槽与分类阶梯），所有色值与 [tokens.json](../../../examples/tokens.json) 保持一致：

```css
:root {
  --font-rounded: ui-rounded, "SF Pro Rounded", "Plus Jakarta Sans", "PingFang SC", system-ui, sans-serif;
  --ease-snappy: cubic-bezier(0.22, 1, 0.36, 1);

  /* 默认：柔珊瑚 Light */
  --canvas: #FAF5F3;
  --surface-1: #FFFFFF;
  --surface-2: #F3EAE7;
  --timeline-rail: rgba(61, 53, 46, 0.11);
  --divider: rgba(61, 53, 46, 0.08);
  --accent-fill: #DE6B62;
  --accent-tint: #C8544B;
  --on-accent: #FFFFFF;
  --accent-soft: rgba(222, 107, 98, 0.11);
  --text-primary: #24201D;
  --text-secondary: #7D756E;
  --text-muted: #B5ADA6;

  /* 固定语义色（不受 data-theme 影响） */
  --semantic-flame: #E56032;
  --semantic-checkin: #329E6A;
  --semantic-trophy: #CE8E16;
  --semantic-destructive: #DC4C46;
}

[data-theme="monochrome"][data-mode="dark"] {
  --canvas: #0C0C0E;
  --surface-1: #18181B;
  --surface-2: #252529;
  --timeline-rail: rgba(255, 255, 255, 0.12);
  --divider: rgba(255, 255, 255, 0.08);
  --accent-fill: #F5F5F7;
  --accent-tint: #F5F5F7;
  --on-accent: #141416;
  --accent-soft: rgba(255, 255, 255, 0.11);
  --text-primary: #F5F5F7;
  --text-secondary: #94949B;
  --text-muted: #56565D;
}

.metric-number {
  font-family: var(--font-rounded);
  font-variant-numeric: tabular-nums;
  font-weight: 700;
}

.structured-card {
  background: var(--surface-1);
  border-radius: 22px;
  border: 0.5px solid rgba(61, 53, 46, 0.05);
  box-shadow: 0 4px 12px rgba(36, 32, 29, 0.05);
}

[data-mode="dark"] .structured-card {
  border-color: rgba(255, 255, 255, 0.06);
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.24);
}
```

---

## 2. React + Framer Motion 核心微交互范式

### 2.1 圆形涟漪勾选框（`RippleCheckCircle`）

```tsx
import { motion, AnimatePresence } from "framer-motion";

export function RippleCheckCircle({
  checked,
  onToggle,
}: {
  checked: boolean;
  onToggle: () => void;
}) {
  return (
    <motion.button
      type="button"
      onClick={onToggle}
      whileTap={{ scale: 0.88 }}
      transition={{ type: "spring", stiffness: 420, damping: 30 }}
      aria-pressed={checked}
      className="relative flex h-11 w-11 items-center justify-center rounded-full"
    >
      <AnimatePresence>
        {checked && (
          <motion.span
            key="ripple"
            initial={{ scale: 0.9, opacity: 0.55 }}
            animate={{ scale: 1.65, opacity: 0 }}
            exit={{ opacity: 0 }}
            transition={{ duration: 0.42, ease: "easeOut" }}
            className="pointer-events-none absolute h-[30px] w-[30px] rounded-full border-2"
            style={{ borderColor: "var(--accent-fill)" }}
          />
        )}
      </AnimatePresence>

      <motion.span
        animate={{
          scale: checked ? [0.78, 1.06, 1] : 1,
          backgroundColor: checked ? "var(--accent-fill)" : "transparent",
          borderColor: checked ? "var(--accent-fill)" : "var(--text-muted)",
        }}
        transition={{ type: "spring", stiffness: 360, damping: 22 }}
        className="flex h-[30px] w-[30px] items-center justify-center rounded-full border-[1.75px]"
      >
        {checked && (
          <motion.svg
            initial={{ scale: 0.4, opacity: 0 }}
            animate={{ scale: 1, opacity: 1 }}
            width="14"
            height="14"
            viewBox="0 0 24 24"
            fill="none"
            stroke="var(--on-accent)"
            strokeWidth="3"
            strokeLinecap="round"
            strokeLinejoin="round"
          >
            <polyline points="20 6 9 17 4 12" />
          </motion.svg>
        )}
      </motion.span>
    </motion.button>
  );
}
```

### 2.2 滑动高亮胶囊（`layoutId`）与悬浮磨砂 Dock

```tsx
<nav
  className="fixed inset-x-4 bottom-2 z-40 mx-auto flex max-w-md items-center gap-1 rounded-full p-1.5 backdrop-blur-xl"
  style={{
    background: "color-mix(in srgb, var(--surface-1) 76%, transparent)",
    border: "0.5px solid var(--divider)",
    boxShadow: "0 6px 20px rgba(0, 0, 0, 0.12)",
  }}
>
  {tabs.map((tab) => {
    const active = tab.id === activeTab;
    return (
      <button
        key={tab.id}
        onClick={() => setActiveTab(tab.id)}
        className="relative flex-1 rounded-full py-2.5 text-xs font-semibold"
        style={{ color: active ? "var(--accent-tint)" : "var(--text-secondary)" }}
      >
        {active && (
          <motion.span
            layoutId="dock-pill"
            transition={{ type: "spring", stiffness: 420, damping: 32 }}
            className="absolute inset-0 rounded-full"
            style={{ background: "var(--accent-soft)" }}
          />
        )}
        <span className="relative z-10">{tab.label}</span>
      </button>
    );
  })}
</nav>
```

---

## 3. Web 端实测防坑清单

1. **移动端 PWA 安全区遮挡**
   - 悬浮 Dock 与主内容容器务必搭配 `padding-bottom: calc(92px + env(safe-area-inset-bottom))`，防止 iOS Safari 底部手势条与悬浮胶囊互相挤压。
2. **极简黑白主题下表单控件反色**
   - 同时声明 CSS `color-scheme: light dark`（随 `data-mode` 切换），确保浏览器原生日期选择器、滚动条不会与暗夜背景冲突。
3. **无障碍降级**
   - 在应用根节点包一层 `<MotionConfig reducedMotion="user">`，同时补充 CSS `@media (prefers-reduced-motion: reduce) { *, *::before, *::after { transition-duration: 0.01ms !important; animation-duration: 0.01ms !important; } }`。
4. **避免硬编码色值**
   - 切勿在 Tailwind 中直接写 `bg-orange-500` / `text-rose-500` 作为主题色——全部绑定 CSS 变量 `bg-[var(--accent-fill)]`，保证 5 套主题切换一次生效。
