# Android (Jetpack Compose / Material 3) 落地指南与防坑清单

本指南将 Structured 的暖调时间轴、全站动态主题与物理微动效映射到原生 **Jetpack Compose**（建议 Compose BOM 2024+ / Kotlin）。

---

## 1. 动态主题色板（`CompositionLocal` + `animateColorAsState`）

不要仅靠 MaterialTheme 的 `primary` 换色——将 `canvas`、`surface1`、`surface2`、`categoryScale` 与独立 `semantic` 统一放进自定义 `StructuredColors` 并通过 `LocalStructuredColors` 提供；配合 `animateColorAsState` 实现切换主题时全站色彩如水墨般平滑过渡。

```kotlin
@Immutable
data class StructuredPalette(
    val canvas: Color,
    val surface1: Color,
    val surface2: Color,
    val timelineRail: Color,
    val divider: Color,
    val accentFill: Color,
    val accentTint: Color,
    val onAccent: Color,
    val accentSoftBg: Color,
    val textPrimary: Color,
    val textSecondary: Color,
    val textMuted: Color,
    val categoryScale: List<Color>
)

@Immutable
object StructuredSemanticColors {
    @Composable fun streakFlame(dark: Boolean) = if (dark) Color(0xFFFF8A5B) else Color(0xE56032)
    @Composable fun checkinGreen(dark: Boolean) = if (dark) Color(0xFF63CA98) else Color(0xFF329E6A)
    @Composable fun trophyGold(dark: Boolean) = if (dark) Color(0xFFF4BE4F) else Color(0xFFCE8E16)
    @Composable fun destructive(dark: Boolean) = if (dark) Color(0xFFF2736D) else Color(0xFFDC4C46)
}

val LocalStructuredPalette = staticCompositionLocalOf<StructuredPalette> {
    error("No StructuredPalette provided")
}
```

---

## 2. 核心微交互组件范式

### 2.1 按压微缩 Modifier

```kotlin
fun Modifier.structuredPressScale(
    interactionSource: MutableInteractionSource,
    targetScale: Float = 0.95f
): Modifier = composed {
    val isPressed by interactionSource.collectIsPressedAsState()
    val scale by animateFloatAsState(
        targetValue = if (isPressed) targetScale else 1f,
        animationSpec = spring(dampingRatio = 0.78f, stiffness = 450f),
        label = "pressScale"
    )
    this.graphicsLayer {
        scaleX = scale
        scaleY = scale
        alpha = if (isPressed) 0.92f else 1f
    }
}
```

### 2.2 圆形涟漪勾选框（`Canvas` + 触觉反馈）

```kotlin
@Composable
fun StructuredCheckCircle(
    checked: Boolean,
    onToggle: () -> Unit,
    modifier: Modifier = Modifier,
    size: Dp = 30.dp
) {
    val palette = LocalStructuredPalette.current
    val haptic = LocalHapticFeedback.current
    val rippleProgress = remember { Animatable(0f) }

    LaunchedEffect(checked) {
        if (checked) {
            rippleProgress.snapTo(0f)
            rippleProgress.animateTo(
                targetValue = 1f,
                animationSpec = tween(durationMillis = 420, easing = FastOutSlowInEasing)
            )
        }
    }

    val fillScale by animateFloatAsState(
        targetValue = if (checked) 1f else 0f,
        animationSpec = spring(dampingRatio = 0.65f, stiffness = 360f),
        label = "checkScale"
    )

    Box(
        modifier = modifier
            .size(44.dp)
            .clickable(
                interactionSource = remember { MutableInteractionSource() },
                indication = null
            ) {
                haptic.performHapticFeedback(HapticFeedbackType.LongPress)
                onToggle()
            },
        contentAlignment = Alignment.Center
    ) {
        Canvas(modifier = Modifier.size(size)) {
            val radius = this.size.minDimension / 2f
            if (rippleProgress.value > 0f && rippleProgress.value < 1f) {
                drawCircle(
                    color = palette.accentFill.copy(alpha = 0.55f * (1f - rippleProgress.value)),
                    radius = radius * (0.9f + 0.75f * rippleProgress.value),
                    style = Stroke(width = 2.dp.toPx())
                )
            }
            if (fillScale > 0.01f) {
                drawCircle(color = palette.accentFill, radius = radius * fillScale)
            } else {
                drawCircle(
                    color = palette.textMuted.copy(alpha = 0.65f),
                    radius = radius,
                    style = Stroke(width = 1.75.dp.toPx())
                )
            }
        }
        AnimatedVisibility(
            visible = checked,
            enter = scaleIn(spring(dampingRatio = 0.65f, stiffness = 360f)) + fadeIn(),
            exit = scaleOut() + fadeOut()
        ) {
            Icon(
                imageVector = Icons.Rounded.Check,
                contentDescription = null,
                tint = palette.onAccent,
                modifier = Modifier.size(15.dp)
            )
        }
    }
}
```

### 2.3 滚轮数字翻滚与滑动胶囊

- **数字滚转**：使用 `AnimatedContent(targetState = value, transitionSpec = { (slideInVertically { -it / 2 } + fadeIn()) togetherWith (slideOutVertically { it / 2 } + fadeOut()) })`，配合 `TextStyle(fontFeatureSettings = "tnum")`。
- **滑动高亮胶囊**：用 `BoxWithConstraints` + `animateDpAsState(targetValue = tabWidth * selectedIndex, animationSpec = spring(0.78f, 450f))` 驱动背景胶囊水平 `offset`。

---

## 3. Android 实测防坑清单

1. **边缘到边缘（Edge-to-Edge）防遮挡**
   - 悬浮底栏容器必须叠加 `WindowInsets.navigationBars.asPaddingValues()`；`LazyColumn` 列表设置 `contentPadding = PaddingValues(bottom = 96.dp + navBarBottom)`（带悬浮 FAB 时增至 `156.dp`）。
2. **圆角一致性与硬件抗锯齿**
   - 大圆角卡片使用 `RoundedCornerShape(22.dp)`，内部控件用 `RoundedCornerShape(14.dp)`；开启 `clipToBounds` 避免子条溢出直角。
3. **无障碍减弱动效**
   - 可读取 `Settings.Global.getFloat(context.contentResolver, Settings.Global.ANIMATOR_DURATION_SCALE, 1f) == 0f` 来跳过额外弹簧动画。
4. **避免强制紫粉 Material You 覆盖极简黑白**
   - 若系统启用 Dynamic Color（Android 12+），当用户选择应用内主题（尤其是极简黑白）时，**必须优先使用应用内 `StructuredPalette`**，不能回退到系统莫奈取色。
