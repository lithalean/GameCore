# GameCore Navigation Context

**Purpose**: Screen flow, user journeys, and state management  
**Version**: 2.0  
**Navigation Pattern**: NavigationPath with Floating Content  
**Last Updated**: 2025-01-27

## Navigation State Machine

```
STATES:
┌─────────────┐     2s      ┌─────────────┐
│   SPLASH    │────────────▶│    TITLE    │
│  (loading)  │             │   (main)    │
└─────────────┘             └──────┬──────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
                    ▼              ▼              ▼
              ┌──────────┐  ┌──────────┐  ┌──────────┐
              │ SETTINGS │  │   LOAD   │  │   GAME   │
              │  PANEL   │  │  PANEL   │  │ CREATOR  │
              └──────────┘  └──────────┘  └──────────┘

TRANSITIONS:
→ Forward: move(edge: .trailing) + opacity
← Back: move(edge: .leading) + opacity  
↓ Modal: fade + scale
↑ Dismiss: fade + scale
```

## Navigation Quick Reference

| From | To | Trigger | Duration | Type | Recovery |
|------|-----|---------|----------|------|----------|
| **Splash** | Title | Auto 2s | 300ms | Fade | N/A |
| **Title** | Creator | New Game | 500ms | Black fade | Back |
| **Title** | Settings | Settings btn | 600ms | Slide left | Back |
| **Title** | Load | Continue btn | 600ms | Slide left | Back |
| **Any** | Title | Back/Cancel | 300ms | Slide right | N/A |
| **Any** | Error | Exception | Instant | None | Retry |

## Navigation Stack Architecture

```swift
// NAVIGATION STATE
NavigationPath: [GameDestination]
├─ .title (root - never removed)
├─ .game 
├─ .settings
└─ .loadGame

// PANEL STATE (local to TitleScreen)
PanelType: enum
├─ .main (default)
├─ .settings
└─ .loadGame

// RELATIONSHIP
NavigationPath controls screens
PanelType controls panels within TitleScreen
```

## Error Recovery Patterns

### Navigation Failure Recovery
```yaml
scenario: "Invalid destination"
detection: navigationDestination throws
recovery:
  1. Log error
  2. Show alert (optional)
  3. Pop to root (.title)
  4. Reset navigation path
code: |
  do {
    try navigate(to: destination)
  } catch {
    navigationPath.removeLast(navigationPath.count)
    showError = true
  }
```

### Transition Interruption Recovery
```yaml
scenario: "User navigates during animation"
detection: Multiple navigation calls
recovery:
  1. Cancel current animation
  2. Complete transition instantly
  3. Start new navigation
  4. No error shown
code: |
  withAnimation(.linear(duration: 0)) {
    completeCurrentTransition()
  }
  startNewNavigation()
```

### Memory Pressure Recovery
```yaml
scenario: "Low memory during navigation"
detection: didReceiveMemoryWarning
recovery:
  1. Cancel non-essential animations
  2. Clear image caches
  3. Complete navigation
  4. Reduce quality if needed
priority: Complete navigation > Visual quality
```

### Background/Foreground Recovery
```yaml
scenario: "App backgrounded during navigation"
detection: scenePhase change
recovery:
  1. Save navigation state
  2. On foreground: restore state
  3. Skip animations
  4. Show current screen
state_preservation: |
  @AppStorage("lastNavigation") 
  var savedPath: Data = Data()
```

## User Journey Flows

### First-Time User Flow
```
START
  │
  ▼
[SPLASH: 2s]
  │
  ▼
[TITLE: Center position]
  │ (800ms delay)
  ▼
[TITLE: Slide up -20pt]
  │ (400ms delay)
  ▼
[BUTTONS: Fade in]
  │
  ├─→ [NEW GAME] → [BLACK FADE] → [CREATOR]
  ├─→ [SETTINGS] → [SLIDE PANEL] → [SETTINGS]
  └─→ [CREDITS] → [FUTURE]
```

### Returning User Flow
```
START
  │
  ▼
[SPLASH: 2s]
  │
  ▼
[TITLE: Final position]
  │
  ├─→ [CONTINUE] → [LOAD PANEL] → [SELECT SLOT] → [CREATOR]
  ├─→ [NEW GAME] → [CREATOR]
  └─→ [SETTINGS] → [SETTINGS PANEL]
```

### Settings Management Flow
```
[MAIN PANEL]
  │
  ▼
[SETTINGS BUTTON]
  │ (600ms slide)
  ▼
[SETTINGS PANEL]
  │
  ├─→ [AUDIO] → [FUTURE: Audio settings]
  ├─→ [GRAPHICS] → [FUTURE: Graphics settings]
  ├─→ [CONTROLS] → [FUTURE: Control mapping]
  └─→ [BACK] → (600ms slide) → [MAIN PANEL]
```

## Navigation Timing Specifications

```yaml
# Screen Transitions
splash_duration: 2000ms
splash_fade: 300ms

title_animation_sequence:
  title_slide_delay: 2000ms
  title_slide_duration: 800ms
  button_fade_delay: 2400ms
  button_fade_duration: 600ms

screen_transitions:
  new_game_sequence:
    button_fade_out: 200ms
    black_overlay_delay: 200ms
    black_overlay_duration: 300ms
    total_before_navigation: 500ms
    
  panel_slide:
    duration: 600ms
    curve: easeInOut
    overlap: 0ms  # No overlap

  back_navigation:
    duration: 300ms
    curve: easeOut
```

## Platform Navigation Adaptations

### iOS Navigation
```swift
// Hide system navigation
.navigationBarHidden(true)
.statusBarHidden(showSplash)

// Gesture support
.gesture(
    DragGesture()
        .onEnded { value in
            if value.translation.width > 50 {
                navigateBack()
            }
        }
)
```

### macOS Navigation
```swift
// Window chrome
.toolbar(.hidden, for: .windowToolbar)
.frame(minWidth: 1024, minHeight: 768)

// Keyboard shortcuts
.keyboardShortcut(.escape) { navigateBack() }
.keyboardShortcut(.return) { confirmAction() }

// Mouse support
.onHover { hovering in
    // Visual feedback
}
```

### tvOS Navigation
```swift
// Focus engine
.focusable()
.focused($focusedButton)
.focusEffect()

// Remote gestures
.onMoveCommand { direction in
    switch direction {
    case .left: navigateBack()
    case .right: navigateForward()
    default: break
    }
}
```

## Navigation State Validation

```yaml
valid_states:
  navigation_path:
    - []  # Root (shows title)
    - [.game]
    - [.settings]
    - [.loadGame]
    
  panel_state:
    - .main
    - .settings
    - .loadGame

invalid_states:
  - Multiple same destinations
  - Empty path with non-title screen
  - Panel state without title screen
  
validation_rules:
  - Title screen always available
  - Only one modal at a time
  - Panels only on title screen
  - Back always goes to previous
```

## Deep Linking Structure

```yaml
# FUTURE IMPLEMENTATION
url_scheme: "gamecore://"

routes:
  home: "gamecore://home"
  game: "gamecore://game"
  new_game: "gamecore://game/new"
  load_game: "gamecore://game/load?slot={1-3}"
  settings: "gamecore://settings"
  settings_audio: "gamecore://settings/audio"
  
parsing: |
  func handleDeepLink(_ url: URL) {
    guard let components = URLComponents(url: url) else { return }
    
    switch components.path {
    case "/game/new":
      navigateToNewGame()
    case "/game/load":
      if let slot = components.queryItems?["slot"] {
        loadGame(slot: slot)
      }
    default:
      navigateToTitle()
    }
  }
```

## Navigation Performance

```yaml
metrics:
  average_transition_time: 485ms
  navigation_memory_spike: 10MB
  animation_fps: 60
  gesture_response: <50ms
  
bottlenecks:
  - Complex view initialization
  - Large image loading
  - Animation calculations
  
optimizations:
  - Preload adjacent screens
  - Cancel interrupted animations
  - Lazy load heavy content
  - Cache navigation destinations
```

## Keyboard & Focus Navigation

### Key Bindings
| Key | Action | Context | Platform |
|-----|--------|---------|----------|
| **Escape** | Navigate back | Any | macOS/tvOS |
| **Enter** | Confirm | Focused | All |
| **Space** | Activate | Button | macOS |
| **Tab** | Next focus | Any | macOS |
| **Arrows** | Move focus | Grid | tvOS |
| **Play/Pause** | Toggle | Media | tvOS |

### Focus Order
```
Title Screen:
1. New Game
2. Continue  
3. Settings
4. Credits

Settings Panel:
1. Audio Settings
2. Graphics Settings
3. Controls
4. Back

Load Panel:
1. Slot 1
2. Slot 2
3. Slot 3
4. Back
```

## Navigation Testing Matrix

### Test Scenarios
```yaml
rapid_navigation:
  description: "Quickly tap between all panels"
  expected: "Smooth transitions, no crashes"
  edge_cases:
    - Double tap
    - Triple tap
    - Opposite directions
    
interrupt_transition:
  description: "Navigate during animation"
  expected: "Complete current, start new"
  edge_cases:
    - Same destination
    - Back during forward
    - Multiple interrupts
    
memory_pressure:
  description: "Navigate with low memory"
  expected: "Complete navigation, reduce quality"
  edge_cases:
    - During transition
    - Multiple screens
    - With background tasks
    
state_restoration:
  description: "Kill app during navigation"
  expected: "Restore to safe state"
  edge_cases:
    - Mid-animation
    - Modal present
    - Deep navigation
```

## Navigation Quick Formulas

```
TRANSITION_TOTAL = fade_out + delay + fade_in
PANEL_POSITION = currentPanel.rawValue * screenWidth
GESTURE_THRESHOLD = screenWidth * 0.3
FOCUS_DISTANCE = abs(currentFocus - targetFocus)
PRELOAD_DEPTH = min(2, availableMemory / 50MB)
```