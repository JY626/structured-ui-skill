# iOS (SwiftUI iOS 17+) 落地指南与防坑清单

本指南提供可直接用于生产级 SwiftUI 应用（Swift 5 / Swift 6 严格并发，iOS 17+，零第三方依赖）的架构模式与已验证防坑清单。

---

## 1. 全站动态主题管理（`@Observable` + `@MainActor`）

将所有主题令牌收敛在一个文件（如 `AppTheme.swift`）中。`ThemeManager.shared` 使用 `@Observable`，当任意视图在 `body` 中读取 `AppTheme.canvas` / `AppTheme.surface1` / `AppTheme.accentFill` 时，SwiftUI 会自动建立观察依赖，切换主题瞬间全站重绘且不丢失导航状态。

```swift
@MainActor
@Observable
final class ThemeManager {
    static let shared = ThemeManager()
    private static let storageKey = "app.accentTheme"

    var selectedTheme: AccentTheme {
        didSet {
            UserDefaults.standard.set(selectedTheme.rawValue, forKey: Self.storageKey)
        }
    }

    init() {
        if let raw = UserDefaults.standard.string(forKey: Self.storageKey),
           let stored = AccentTheme(rawValue: raw) {
            self.selectedTheme = stored
        } else {
            self.selectedTheme = .coralRose
        }
    }
}

enum AppTheme {
    @MainActor static var canvas: Color { ThemeManager.shared.selectedTheme.canvas }
    @MainActor static var surface1: Color { ThemeManager.shared.selectedTheme.surface1 }
    @MainActor static var surface2: Color { ThemeManager.shared.selectedTheme.surface2 }
    @MainActor static var accentFill: Color { ThemeManager.shared.selectedTheme.accentFill }
    @MainActor static var accentTint: Color { ThemeManager.shared.selectedTheme.accentTint }
    @MainActor static var onAccent: Color { ThemeManager.shared.selectedTheme.onAccent }
    @MainActor static var completedRowTint: Color { ThemeManager.shared.selectedTheme.completedRowTint }

    // 独立固定语义色（不随主题变化）
    static let streakFlame = Color.adaptive(dark: 0xFF8A5B, light: 0xE56032)
    static let checkinGreen = Color.adaptive(dark: 0x63CA98, light: 0x329E6A)
    static let trophyGold = Color.adaptive(dark: 0xF4BE4F, light: 0xCE8E16)
    static let destructive = Color.adaptive(dark: 0xF2736D, light: 0xDC4C46)
}
```

---

## 2. 核心微交互组件范式

### 2.1 按压微缩与圆形涟漪勾选框

```swift
struct StructuredPressButtonStyle: ButtonStyle {
    @Environment(\.accessibilityReduceMotion) private var reduceMotion
    var scale: CGFloat = 0.95

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed && !reduceMotion ? scale : 1)
            .opacity(configuration.isPressed ? 0.92 : 1)
            .animation(reduceMotion ? nil : .spring(response: 0.30, dampingFraction: 0.78), value: configuration.isPressed)
    }
}

struct StructuredCheckCircle: View {
    @Environment(\.accessibilityReduceMotion) private var reduceMotion
    let isCompleted: Bool
    var size: CGFloat = 30
    let action: () -> Void

    @State private var rippleScale: CGFloat = 0.85
    @State private var rippleOpacity: Double = 0

    var body: some View {
        Button(action: action) {
            ZStack {
                Circle()
                    .stroke(AppTheme.accentFill, lineWidth: 2)
                    .frame(width: size, height: size)
                    .scaleEffect(rippleScale)
                    .opacity(rippleOpacity)

                if isCompleted {
                    Circle()
                        .fill(AppTheme.accentFill)
                        .frame(width: size, height: size)
                        .transition(.scale(scale: 0.65).combined(with: .opacity))

                    Image(systemName: "checkmark")
                        .font(.system(size: size * 0.44, weight: .bold))
                        .foregroundStyle(AppTheme.onAccent)
                        .symbolEffect(.bounce, value: isCompleted)
                } else {
                    Circle()
                        .strokeBorder(AppTheme.textMuted.opacity(0.65), lineWidth: 1.75)
                        .frame(width: size, height: size)
                }
            }
            .frame(width: max(size + 8, 44), height: max(size + 8, 44))
            .contentShape(Circle())
        }
        .buttonStyle(StructuredPressButtonStyle(scale: 0.88))
        .animation(reduceMotion ? nil : .spring(response: 0.36, dampingFraction: 0.66), value: isCompleted)
        .sensoryFeedback(.success, trigger: isCompleted)
        .onChange(of: isCompleted) { _, completed in
            guard completed, !reduceMotion else { return }
            rippleScale = 0.9
            rippleOpacity = 0.55
            withAnimation(.easeOut(duration: 0.42)) {
                rippleScale = 1.65
                rippleOpacity = 0
            }
        }
    }
}
```

### 2.2 悬浮磨砂底栏 + 状态保留切页 + 二级页隐藏

```swift
ZStack(alignment: .bottom) {
    ZStack {
        tabLayer(for: .home) { HomeView(isTabBarHidden: $hideForHome) }
        tabLayer(for: .calendar) { CalendarView(isTabBarHidden: $hideForCalendar) }
    }
    .animation(reduceMotion ? nil : .easeOut(duration: 0.25), value: selectedTab)

    if !isDockHidden {
        floatingDock
            .padding(.horizontal, 18)
            .padding(.bottom, 8)
            .transition(.move(edge: .bottom).combined(with: .opacity))
    }
}
```

---

## 3. iOS 实测防坑清单（必读）

1. **二级页隐去底栏千万不要把 `.onDisappear` 挂在 `NavigationStack` 内部根视图上**
   - *坑点*：当 `NavigationStack` Push 进入子页面时，根内容视图会触发 `.onDisappear`，若在里面把 `isTabBarHidden` 重置为 `false`，会导致底部悬浮导航栏在详情页突然重新弹出遮挡底部按钮！
   - *正解*：把 `.onChange(of: navigationPath.isEmpty, initial: true) { _, isEmpty in isTabBarHidden = !isEmpty }` 挂在外层 `NavigationStack` 自身上，并删掉内层 `.onDisappear` 重置。
2. **`.contentTransition(.numericText(value:))` 类型要求**
   - 必须传入 `Double`（如 `Double(reps)`），直接传 `Int` 会触发编译报错；倒计时请使用 `.contentTransition(.numericText(countsDown: true))`。
3. **视图结构默认参数不可引用 `@MainActor` 动态令牌**
   - 若 `AppTheme.surface2` 标记为 `@MainActor`，则不能写 `var trackColor: Color = AppTheme.surface2`，需改为 `var trackColor: Color? = nil` 并在 `body` 中 `trackColor ?? AppTheme.surface2`。
4. **滚动容器底部安全留白**
   - 在 iOS 17+ 的 `List` / `ScrollView` 上使用 `.contentMargins(.bottom, 92, for: .scrollContent)`（含悬浮 `+` FAB 的列表可留 `92 + 64`），避免 iPhone SE 等紧凑机型末行内容被悬浮底栏遮挡。
5. **`#Preview` 预览防崩溃**
   - 所有用到 `@Environment(ThemeManager.self)` 的预览均注入 `.environment(ThemeManager.shared)`；子页接收 `@Binding var isTabBarHidden` 时提供默认 `init(isTabBarHidden: Binding<Bool> = .constant(false))`。
