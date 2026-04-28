# Mobile and Cross-Platform

## Mobile — Universal Rules

These apply to every mobile platform (iOS, Android, React Native, mobile web):

- **Touch targets**: 44pt (iOS) / 48dp (Android) minimum for all interactive elements. Never under.
- **Safe areas**: account for status bar, notch, home indicator, and keyboard insets. Use `SafeAreaView` in RN, `env(safe-area-inset-*)` on web.
- **Keyboard-aware forms**: scroll content above keyboard. Use `KeyboardAvoidingView` (RN) or `position: fixed` footer with `resize` listener on web.
- **Thumb reach**: primary actions in bottom 40% of screen. Destructive/secondary in top half or overflow menus.
- **Bottom tabs**: for top-level navigation (max 5 tabs). No hamburger menus.
- **Stack navigation**: for depth. Swipe-back on iOS must be preserved.
- **No hover-only controls**: every interaction must have a touch equivalent.
- **Body text**: minimum 15px. Target 16–17px for readability.
- **No horizontal overflow**: test at 375px and 430px.
- **Offline/retry**: mandatory for field apps, delivery apps, or any app used on unreliable networks.

## iOS

- **Large titles** on root screens; inline titles on pushed screens.
- **Tab bars** at the bottom; standard spacing and icon sizing.
- **Sheets** for contextual actions (half-sheet or full-sheet). Dismissible by swipe or button.
- **Action sheets** for multiple choice destructive/confirmation actions (`.actionSheet`).
- **Swipe-back** via `UINavigationController` / `react-native-screens` native stack. Never disable it.
- **SF Symbol–inspired icons**: consistent weight and optical sizing.
- **Dynamic Type**: text scales with system font size setting. Never hardcode font sizes in points without scaling.
- **Haptics**: `UIImpactFeedbackGenerator` / `expo-haptics`. Use on: button press (light), toggle (medium), destructive action (heavy), success completion (notification success).

### iOS Animation Patterns
- Sheet appearance: spring up from bottom, backdrop fades in.
- Sheet dismiss: spring down, backdrop fades out.
- Navigation push: new screen slides in from right (standard iOS behavior).
- Navigation pop: screen slides out to right.
- Tab switch: instant content swap (no animation between tabs — iOS convention).
- Context menu appear: scale from touch point + blur background.
- Long-press picker: springy appear at touch point.
- Pull to refresh: spinner spins, content translates down.

## Android

- **Navigation patterns**: Material 3 nav bar (bottom, ≤5 items) or nav drawer for complex apps.
- **System back**: support gesture navigation (Android 10+) and back button. Never swallow back events without reason.
- **FAB**: only for one single dominant creation action per screen. Never multiple FABs.
- **Snackbars**: for temporary feedback (not toasts — use Snackbar API). Support undo action in snackbar.
- **Bottom sheets**: for contextual actions, share sheets, filter pickers.
- **Ripple effect**: every touchable element must have a Material ripple (`Ripple` in RN).
- **Dynamic color (Material You)**: follow system color scheme when targeting Android 12+.

### Android Animation Patterns
- Navigation push: fade + slight Y translate up (Material 3 default). Not horizontal slide.
- Navigation pop: reverse fade + Y.
- FAB: scale in on screen enter, scale out on scroll down (if using scroll-away pattern).
- Bottom sheet appear: slide up from bottom edge.
- Shared element: scale + position transition between list item and detail screen.
- Ripple: radial expansion from touch point on all interactive elements.

## Mobile Web

- No fixed desktop sidebars. Use bottom drawer or collapsible overlay.
- Dense tables collapse to detail cards only when comparison is not central to the task.
- Filters: accessible via bottom sheet or slide-down panel — not hidden in a sidebar.
- Body text ≥ 14px. Target 16px.
- Test at 375px (iPhone SE) and 430px (iPhone Pro Max).
- Viewport meta tag: `width=device-width, initial-scale=1`.
- Avoid `vh` for full-height layouts — use `dvh` or `100svh` to handle mobile browser chrome.
- Avoid fixed position elements that conflict with virtual keyboard.

## React Native / Expo

### Navigation
- Use `react-navigation` with `react-native-screens` and native stack.
- Tab navigator for top-level; stack navigator for depth.
- Do not use custom JS-based navigation animations for main flows — use native stack.
- Pass params via navigation, not global state.

### Gesture and Animation
- **Reanimated 2** is mandatory for any gesture-driven or 60fps animation.
- Use `useSharedValue`, `useAnimatedStyle`, `withSpring`, `withTiming`, `withSequence`.
- Use `react-native-gesture-handler` for all gesture recognizers. Never use `TouchableOpacity` for swipe gestures.
- Gesture must follow finger 1:1 with no lag. Spring back to snap point on release.

#### Common RN Gesture Patterns
```js
// Swipe to delete
const translateX = useSharedValue(0);
const gestureHandler = useAnimatedGestureHandler({
  onActive: (event) => { translateX.value = event.translationX; },
  onEnd:    (event) => {
    if (event.translationX < -THRESHOLD) {
      translateX.value = withSpring(-ACTION_WIDTH);
    } else {
      translateX.value = withSpring(0);
    }
  },
});

// Bottom sheet drag to snap points
const translateY = useSharedValue(COLLAPSED_Y);
// Snap to: EXPANDED_Y, HALF_Y, COLLAPSED_Y
```

Spring config starting point:
```js
withSpring(value, { mass: 1, damping: 20, stiffness: 250 })
```

### Haptics
- Button press: `Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)`
- Toggle on: `Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)`
- Destructive/delete: `Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Heavy)`
- Success: `Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)`
- Error: `Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error)`

### Layout
- Use `flex` for all layouts. No fixed pixel dimensions except icons and media.
- `StyleSheet.create()` for all styles — not inline objects.
- Respect `useSafeAreaInsets()` for padding.
- Use `FlatList` or `FlashList` for long lists. Never `ScrollView` with mapped items.
- Use `KeyboardAvoidingView` with `behavior="padding"` (iOS) / `"height"` (Android).

### Platform-Specific Code
```js
import { Platform } from 'react-native';
const shadowStyle = Platform.select({
  ios:     { shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.12, shadowRadius: 8 },
  android: { elevation: 4 },
});
```

## Cross-Platform (Web + Mobile)

### Shared Design Tokens
Define tokens once and map to platform equivalents:
- CSS variables for web.
- JS constants / React Native StyleSheet tokens for native.
- A single `tokens.ts` or `design-tokens.json` as source of truth.

### Platform-Specific Navigation
- Do not use a single navigation model for all platforms.
- Web: top nav or sidebar.
- iOS: tab bar + native stack.
- Android: bottom nav + native stack.

### Adaptive Layout Strategy
- Mobile: single column, bottom nav, stacked sections.
- Tablet: optional split-pane, collapsible sidebar.
- Desktop: persistent sidebar, multi-column, dense tables.
- Use `useWindowDimensions` (RN) or CSS container queries (web) — not hardcoded breakpoints.

## Electron / Tauri (Desktop App)

- Implement menu bar (File, Edit, View, Help, app-specific).
- Command palette (`⌘K` / `Ctrl+K`) for all power actions.
- Status bar at the bottom: connection, sync status, user, app state.
- Window controls awareness: don't place content under traffic lights (macOS) or title bar.
- File/resource tree in a resizable left panel.
- Tab bar for multi-document or multi-workspace.
- Full context menus: files, rows, tabs, canvas objects, selected text.
- Keyboard shortcuts for every frequent action.
- Drag-and-drop for files, tabs, items.

## PWA

- Install prompt: only when the user has engaged meaningfully. Never on first visit.
- Offline state: show clearly when offline. Cache critical UI and data with service worker.
- Update available: non-blocking banner with "Update now" action.
- App shell loading: skeleton that appears before JS hydrates.
- Push notification permissions: ask with clear rationale, never speculatively.
- `manifest.json`: correct icons (192, 512), theme-color, display, start_url.
