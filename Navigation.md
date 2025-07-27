# GameCore Navigation Context

**Navigation Pattern**: NavigationPath with Floating Content  
**State Management**: Observable GameStateManager  
**Last Updated**: January 2025

## Navigation Architecture

### Screen Hierarchy
```
Application Root
├── SplashScreen (2s auto-dismiss)
└── MainGameView
    ├── Persistent Background Layer
    │   ├── TitleBackground (always visible)
    │   └── TitleScreenView (always visible)
    │
    └── Floating Content Layer
        ├── TitleScreenContent
        │   ├── MainPanel
        │   ├── SettingsPanel
        │   └── LoadGamePanel
        ├── CreatorScreen
        ├── SettingsScreen (future)
        └── LoadGameScreen (future)
```

### Navigation Destinations
```swift
enum GameDestination: Hashable {
    case title      // Title screen (default)
    case game       // Creator interface
    case settings   // Global settings
    case loadGame   // Save management
}
```

## State Management

### GameStateManager
```swift
@Observable class GameStateManager {
    // Core navigation state
    var navigationPath = NavigationPath()
    
    // Navigation methods
    func navigateToGame()      // -> CreatorScreen
    func navigateToSettings()  // -> SettingsScreen
    func navigateToLoadGame()  // -> LoadGameScreen
    func navigateBack()        // Pop current screen
}
```

### Panel State Management
```swift
// Internal to TitleScreen
enum PanelType {
    case main      // New Game, Continue, Settings, Credits
    case settings  // Audio, Graphics, Controls, Back
    case loadGame  // Slot 1, Slot 2, Slot 3, Back
}

// Managed locally within TitleScreen
@State private var currentPanel: PanelType = .main
```

## Navigation Flow Patterns

### Application Launch
```
1. App Start
   └── SplashScreen (2s)
       └── Fade transition
           └── MainGameView
               └── TitleScreenContent (default)
```

### Title Screen Navigation
```
TitleScreenContent (Main Panel)
├── New Game → Black transition → CreatorScreen
├── Continue → LoadGamePanel (slide animation)
├── Settings → SettingsPanel (slide animation)
└── Credits → Print action (future implementation)

Settings Panel
├── Audio Settings → Print action
├── Graphics Settings → Print action
├── Controls → Print action
└── Back to Main → MainPanel (slide animation)

Load Game Panel
├── Slot 1-3 → Print action
└── Back to Main → MainPanel (slide animation)
```

### Screen Transition Patterns

#### New Game Transition
```swift
Timeline:
0.0s: User taps "New Game"
0.2s: Buttons fade out (opacity 1.0 → 0.0)
0.2s: Black overlay begins fade in
0.5s: Navigation triggered
0.6s: CreatorScreen appears over persistent background
```

#### Panel Slide Transitions
```swift
Duration: 0.6s
Animation: .easeInOut
Direction: 
  - Forward: .move(edge: .trailing) + .opacity
  - Back: .move(edge: .leading) + .opacity
```

## Navigation Rules

### Persistent Background Rules
1. Background layers NEVER animate or change
2. Background created once at app launch
3. Background visible through all floating content
4. No navigation affects background state

### Floating Content Rules
1. Only one floating screen active at a time
2. Transitions animate floating layer only
3. All screens respect persistent background
4. Navigation managed by GameStateManager

### Platform-Specific Navigation
```swift
// iOS Navigation
.navigationBarHidden(true)
.statusBarHidden(showSplash)

// macOS Navigation  
.toolbar(.hidden, for: .windowToolbar)
.frame(minWidth: 1024, minHeight: 768)

// tvOS Navigation
.navigationBarHidden(true)
.ignoresSafeArea(.all)
```

## User Journey Maps

### First-Time User
```
1. Launch → Splash (2s)
2. Title appears centered
3. Title slides up (-20pt)
4. Buttons slide in from right
5. User explores panels
6. Selects "New Game"
7. Transition to Creator
```

### Returning User
```
1. Launch → Splash (2s)
2. Skip to button state
3. Select "Continue" 
4. Load Game panel slides in
5. Select save slot
6. Load into Creator with saved state
```

### Settings Flow
```
1. From Main → Settings button
2. Settings panel slides in
3. Adjust preferences
4. Back to Main
5. Settings applied globally
```

## Navigation State Preservation

### Current Implementation
- Navigation path persisted during session
- Panel states reset on screen change
- No deep linking support yet

### Future Enhancements
```swift
// Planned state preservation
struct NavigationState: Codable {
    let currentScreen: GameDestination
    let panelState: PanelType?
    let savedGameSlot: Int?
    let timestamp: Date
}
```

## Transition Specifications

### Screen-Level Transitions
| From | To | Duration | Type |
|------|-----|----------|------|
| Splash | Title | 0.3s | Fade |
| Title | Creator | 0.5s | Fade + Black |
| Any | Previous | 0.3s | Slide |

### Panel-Level Transitions
| From | To | Duration | Type |
|------|-----|----------|------|
| Main | Settings | 0.6s | Slide left |
| Main | LoadGame | 0.6s | Slide left |
| Settings | Main | 0.6s | Slide right |
| LoadGame | Main | 0.6s | Slide right |

## Error Handling

### Navigation Failures
```swift
// Graceful fallback pattern
do {
    try navigateToDestination()
} catch {
    // Return to safe state (Title)
    navigationPath.removeLast(navigationPath.count)
}
```

### Invalid States
- Empty navigation path → Show Title
- Unknown destination → Show Title
- Transition interrupted → Complete transition

## Keyboard Navigation (macOS/tvOS)

### Key Bindings
- **Escape**: Navigate back / Close panel
- **Enter**: Confirm selection
- **Tab**: Next focusable element
- **Arrow Keys**: Navigate between buttons

### Focus Management
```swift
// tvOS focus guide
.focusable()
.focusEffect()
.focused($focusedButton)

// macOS keyboard navigation
.keyboardShortcut(.return, modifiers: [])
.keyboardShortcut(.escape, modifiers: [])
```

## Deep Linking Structure (Future)

### Planned URL Scheme
```
gamecore://navigate/[destination]
├── gamecore://navigate/title
├── gamecore://navigate/game
├── gamecore://navigate/settings
└── gamecore://navigate/load?slot=1
```

## Performance Considerations

### Navigation Optimization
- Lazy loading of destination views
- Preload adjacent panels for smooth slides
- Cancel animations if interrupted
- Minimal state during transitions

### Memory Management
- Previous screens not retained
- Only active panel content in memory
- Persistent background shared instance
- Navigation path lightweight storage

## Testing Scenarios

### Navigation Test Cases
1. **Rapid Navigation**: Quickly tap between panels
2. **Interrupt Transitions**: Navigate during animation
3. **Deep Stack**: Navigate through all screens
4. **Memory Pressure**: Navigate under low memory
5. **Background/Foreground**: Navigation state preservation
6. **Platform Switching**: Test all platforms

### Edge Cases
- Double-tap navigation buttons
- Navigate during screen transition
- Force quit during navigation
- Network loss during future online features