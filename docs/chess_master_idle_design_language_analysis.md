# Lichess Flutter design + implementation pattern analysis for `chess_master_idle`

## Scope and current limitation

This first pass analyzes the **Lichess Flutter repo** and extracts patterns that can be adapted into your game UI language.

I could not locate your `learntechgames-monorepo` / `app/chess_master_idle` workspace in this container, so this report focuses on the source side (Lichess) and provides a migration blueprint you can apply once that repo is available locally.

---

## 1) Design-system backbone patterns in Lichess

### 1.1 Token-first styling (color, spacing, radius, typography)

Lichess centralizes reusable visual constants into style/token-like files:

- `LichessColors` defines semantic color families (`primary`, `secondary`, `accent`, `error`, `brag`, etc.).
- `Styles` defines text presets, spacing presets, border radii, and helper color transforms (`darken`, `lighten`).
- Theme extensions (`LichessCustomColors`, `CustomTheme`) layer app-specific semantic roles on top of Material `ThemeData`.

**Why this matters for your language:** this is directly compatible with your “Surface / Depth / Meaning” pillars. Build your game UI kit from semantic tokens, not ad-hoc widget colors.

### 1.2 Theme generated from gameplay context

Lichess derives app theme seed from board preferences and user settings, then constructs color schemes with neutralized surfaces + harmonized accents.

This is powerful for games: it keeps identity coherent while allowing personalization and contextual mood shifts.

### 1.3 Platform-adaptive shell without forking UX logic

Lichess uses platform-aware wrappers (`PlatformWidget`, `PlatformScaffold`, `PlatformAppBar`) so behavior and feel can adjust per OS while keeping feature logic shared.

**Adaptation for chess_master_idle:** keep one gameplay UX model, then tune interaction polish (blur, nav bars, haptics, control density) per platform.

---

## 2) Motion + feedback patterns worth adapting

### 2.1 Intentional micro-interactions

Small but consistent tap feedback appears in reusable controls:

- `OpacityButton`: pressed-state fade (`AnimatedOpacity`, 100ms).
- `BoardThumbnail`: tap-down scale (`AnimatedScale`, 100ms).

These align with your “No silent interaction” rule.

### 2.2 Continuous-state feedback loops (loading, refresh, hold)

Lichess has practical utility patterns:

- `Shimmer` + `ShimmerLoading` for skeleton states.
- `HapticRefreshIndicator` for iOS refresh confirmation.
- `RepeatButton` for accelerated long-press actions with haptic kick.

For idle games, this pattern is gold: apply to upgrade buttons, batch-buy, auto-collect, and async reward claims.

### 2.3 Transition primitives for information density

`ExpandedSection` uses a controlled `SizeTransition` (300ms), while board/position widgets expose animation durations as parameters.

This is exactly the sort of controlled motion budget your taxonomy calls for: motion by role, not decorative noise.

---

## 3) Component architecture patterns to port

### 3.1 Domain widgets over generic primitives

Lichess ships domain components like `BoardWidget`, `GameLayout`, `StatCard`, `ProgressionWidget`, and board thumbnails.

**Pattern:** encapsulate domain semantics into reusable widgets with clear interfaces.

For your project, introduce semantic widgets immediately:

- `GameButton`, `GamePanel`, `GameBadge`, `GameProgressBar`, `GameStatTile`, `GameCurrencyCounter`, `GameBoardFrame`.

### 3.2 State ownership + persistence strategy

Lichess pairs Riverpod notifiers with serializable preference models (`freezed` + JSON converters), so visual settings are first-class and persist cleanly.

For your design language rollout, create durable state for:

- theme family
- motion intensity
- haptic level
- accessibility contrast mode
- reduced FX mode

### 3.3 Layout system that respects gameplay first

`GameLayout` is board-centric and adapts orientation/tablet behavior with explicit constraints.

Use this principle in idle chess UI too: gameplay/info hierarchy should drive layout breakpoints, not generic responsive templates.

---

## 4) Mapping Lichess patterns to your 6-pillar language

## Surface

- Lift `Styles` + `LichessColors` approach into semantic game tokens.
- Add material presets (e.g., alloy/glass/arcade) as surface variants.

## Depth

- Reuse layered container strategy (`background`, `surfaceContainer`, shadows/radius).
- Define depth levels (`z0..z4`) mapped to panel, action, priority, overlay.

## State

- Use reusable stateful wrappers like Lichess button patterns.
- Expand from idle/pressed/disabled to your full state matrix (selected, cooldown, warning, success, failure).

## Motion

- Keep motion durations tokenized (100ms, 200ms, 300ms, etc.) and tied to intent.
- Prefer localized micro-animations over large global motion.

## Feedback

- Copy the haptic + shimmer + progression idiom.
- Add animated number ticks and reward pulses for idle loops.

## Hierarchy

- Use typography tiers and color semantics to create clear visual rank.
- Reserve heavy motion/glow for economy events, unlocks, and urgency states.

---

## 5) Recommended Flutter design-language package structure for `chess_master_idle`

```text
app/chess_master_idle/lib/ui/
  tokens/
    game_colors.dart
    game_spacing.dart
    game_radius.dart
    game_typography.dart
    game_motion.dart
    game_elevation.dart
  theme/
    game_theme.dart
    game_theme_extensions.dart
  state/
    ui_preferences.dart
    ui_preferences_notifier.dart
  components/
    buttons/game_button.dart
    panels/game_panel.dart
    feedback/game_shimmer.dart
    feedback/game_haptic.dart
    progress/game_progress_bar.dart
    economy/currency_counter.dart
    stats/stat_tile.dart
```

This mirrors Lichess strengths: centralized tokens + semantic widgets + persistent preferences.

---

## 6) First implementation pass (suggested sequence)

1. **Create token layer** (color roles, type roles, radius, shadow, motion durations/curves).
2. **Create `ThemeExtension`s** for game-only semantics (reward, warning pulse, rarity).
3. **Build 5 core semantic widgets** (`GameButton`, `GamePanel`, `GameProgressBar`, `GameBadge`, `CurrencyCounter`).
4. **Implement interaction states** (idle/hover/pressed/disabled/selected + optional success/failure).
5. **Add feedback utilities** (haptic wrapper, shimmer loader, animated number counter).
6. **Migrate one vertical slice** (e.g., “upgrade panel”) to validate consistency.
7. **Add reduced-motion and high-contrast toggles** in persisted prefs.

---

## 7) What to collect next from your repo (once available)

To produce the direct Lichess -> `chess_master_idle` migration map, I need:

- Existing UI architecture (`theme`, `widgets`, `screens` folders).
- Current state management approach.
- Existing color and typography constants.
- Any animation helpers and duration constants.
- 2–3 representative screens (HUD, upgrades/shop, meta progression).

With that, I can produce:

- exact token mapping table,
- component-by-component replacement plan,
- and concrete refactor PR-ready tasks.
