# GameCore — AI Context Document

**Date**: January 2025  
**Status**: Active Development - Core UI Framework Complete  
**Purpose**: Accurate context for AI systems working with GameCore codebase

---

## Project Identity

**GameCore** is a modular boilerplate framework for native Apple game development that provides everything except the game engine itself.

**Key Facts:**
- Part of larger Orchard ecosystem (RealityAssets, RealitySyntax, GameCore)
- Cross-platform (iOS, macOS, tvOS)
- Engine-agnostic architecture
- SwiftUI-based with RealityKit integration
- Maximum modularity design philosophy
- Apple Silicon optimized

---

## Current Architecture

### Directory Structure
```
GameCore/
├── Styles/
│   ├── TitleBackground.swift          # Gradient background component
│   ├── TitleGridView.swift           # 3D RealityKit grid overlay
│   ├── MenuButton.swift              # Modern menu button with cross-platform haptics
│   └── TitleButton.swift             # Styled game buttons (primary/secondary/standard)
├── Views/
│   ├── MainGameView.swift            # Root navigation container with NavigationStack
│   ├── TitleScreen.swift             # Animated title screen with 5-track sliding panels
│   ├── EnhancedGameView.swift        # In-game UI wrapper with menu integration
│   ├── GameSettingsView.swift        # Settings interface
│   ├── GameLoadView.swift            # Save/load interface
│   ├── CreatorScreen.swift           # Game creator interface with windowed layout
│   └── SplashScreen.swift            # Initial app loading screen
├── Managers/
│   ├── GameStateManager.swift        # Navigation and state management with NavigationPath
│   └── SlidingPanelManager.swift     # Panel animation system (unused in current implementation)
├── GameCore/Core/
│   └── TitleGrid.swift               # RealityKit Entity generation for 3D grid
├── Styles/
│   ├── GameMenuSystem.swift          # 3-box modular menu system
│   ├── GameMenuTop.swift             # Top menu bar component
│   ├── GameMenuCenter.swift          # Center content area component
│   └── GameMenuBottom.swift          # Bottom menu bar component
└── App/
    └── SplashScreen.swift            # Initial app loading screen
```

### Component Dependencies
- TitleScreen depends on: TitleBackground, TitleGridView, TitleButton
- MainGameView depends on: TitleScreen, GameStateManager, CreatorScreen
- TitleGridView depends on: TitleGrid (RealityKit Entity)
- CreatorScreen depends on: TitleBackground, TitleGridView, GameMenuSystem
- GameMenuSystem depends on: GameMenuTop, GameMenuCenter, GameMenuBottom
- All components use cross-platform navigation macros

---

## Current Implementation Status

### Working Components
- **TitleScreen.swift**: Complete 5-track sliding panel system with animations
- **TitleBackground.swift**: Gradient background component (black to blue/purple)
- **TitleGridView.swift**: RealityKit 3D grid with action RPG camera perspective
- **TitleGrid.swift**: RealityKit Entity generation with platform-specific colors
- **GameStateManager.swift**: NavigationPath-based state management
- **MainGameView.swift**: Root navigation with NavigationStack implementation
- **MenuButton.swift**: Cross-platform button with modern sensory feedback
- **CreatorScreen.swift**: Windowed game interface with centered overlay
- **GameMenuSystem.swift**: Modular 3-box layout system
- **Cross-platform navigation**: Conditional compilation for iOS/macOS/tvOS

### Key Features Implemented
- Title animation: slides from center to top (-20 offset) while buttons slide in from right
- 5-track button layout: positions 1,2,3,spacer,5 with consistent alignment across panels
- Sliding panel system: main/settings/loadGame panels with asymmetric transitions
- RealityKit integration: camera at [0,15,35] looking at [0,0,0] for action RPG perspective
- Cross-platform navigation: iOS navigationBarHidden, macOS toolbar hidden, tvOS navigationBarHidden
- Screen transitions: Fade to black transition between title and creator screens
- Windowed interface: GameMenuSystem overlays on background with constrained sizing

### Animation System Implementation
```swift
// Current animation implementation in TitleScreen
titleOffset: starts at 0, animates to -20 after 2 second delay
panelTransitions: uses .asymmetric with .move(edge: .trailing/.leading) + .opacity
slideToPanel(): withAnimation(.easeInOut(duration: 0.6))

// Screen transition implementation in MainGameView
startNewGame(): Quick button fade (0.2s) -> black transition (0.3s) -> navigate to CreatorScreen
```

### Component Integration Pattern
```swift
// How components actually connect
ZStack {
    TitleBackground()    // Gradient component (no dependencies)
    TitleGridView()      // RealityKit component (depends on TitleGrid)
    // UI layer here
}

// CreatorScreen windowed layout
ZStack {
    TitleBackground()    // Full screen background
    TitleGridView()      // Full screen 3D grid
    GameMenuSystem()     // Centered overlay (800x600)
        .background(Color.black.opacity(0.8))
        .cornerRadius(12)
        .shadow(...)
}
```

---

## 3D Grid Implementation

### RealityKit Integration
- Uses RealityView with PerspectiveCamera
- Camera positioned at [0, 15, 35] for action RPG angle
- Grid generated via TitleGrid.makeGridEntity()
- Platform-specific color handling (UIColor/NSColor)

### Grid Specifications
- 100x100 grid with 1.0 spacing
- Red chunk borders at edges
- White major lines every 10 units
- Gray minor lines with 0.3 alpha
- Text labels with billboard components
- Origin marker in yellow
- X/Z axis labels in red/blue

### Performance Considerations
- Lazy entity generation
- Billboard text for readability
- Unlit materials for performance
- Platform-optimized font loading

---

## **CRITICAL ISSUES - REALITYKIT MEMORY LEAKS & CRASHES**

### **Known Problems**
- **macOS Freeze**: App locks up and requires force quit shortly after CreatorScreen loads
- **Memory Leak**: RealityKit entities not properly deallocated when views switch
- **Performance Degradation**: Multiple grid instances accumulate in memory
- **Navigation Issues**: Sometimes appears to go back to splash screen instead of forward

### **Root Cause Analysis**
- **RealityKit Memory Management**: Known issue with ARView/RealityView not properly deallocating
- **Entity Accumulation**: TitleGrid.makeGridEntity() creates ~400+ entities (lines + labels + base plane)
- **No Cleanup**: No proper entity removal when views disappear
- **Multiple Instances**: Grid recreated on each view appearance without cleanup

### **Attempted Solutions (DO NOT REPEAT)**
- ❌ **Entity state management**: Caused grid to disappear
- ❌ **onDisappear cleanup**: Still leaked memory
- ❌ **SwiftUI Canvas replacement**: Not what was requested
- ❌ **Reduced grid complexity**: Changed original design intent
- ❌ **Conditional grid loading**: Added unnecessary complexity

### **Current Workarounds**
- Grid works perfectly on TitleScreen
- Issue only occurs when transitioning to CreatorScreen
- Problem likely related to multiple RealityKit instances running simultaneously

### **Technical Details**
```swift
// Current TitleGrid implementation (CAUSES CRASHES)
- 100x100 grid (10,000 lines)
- ~200 text labels with billboard components
- 1 base plane
- 3 axis markers
- Total: ~400+ RealityKit entities per grid instance

// Memory leak pattern:
TitleScreen (Grid Instance 1) -> CreatorScreen (Grid Instance 2) -> CRASH
```

### **DO NOT ATTEMPT**
- Do not modify TitleGrid.swift without explicit instructions
- Do not add entity cleanup code
- Do not replace RealityKit with SwiftUI Canvas
- Do not reduce grid complexity
- Do not add state management to TitleGridView
- Do not create fallback grid systems

---

## Cross-Platform Architecture

### Navigation Pattern
```swift
#if os(iOS)
.navigationBarHidden(true)
#elseif os(macOS)
.toolbar(.hidden, for: .windowToolbar)
#elseif os(tvOS)
.navigationBarHidden(true)
#endif
```

### Color System
Uses platform-specific color types:
- macOS: NSColor
- iOS/tvOS: UIColor
- Conditional compilation throughout

### Input Handling
- MenuButton uses .sensoryFeedback() for modern haptics
- Cross-platform gesture support
- Platform-appropriate button styles

---

## State Management

### GameStateManager
```swift
@Observable class GameStateManager {
    var navigationPath: NavigationPath = NavigationPath()
    
    func navigateToGame() // Pushes .game destination -> CreatorScreen
    func navigateToSettings() // Pushes .settings destination  
    func navigateToLoadGame() // Pushes .loadGame destination
    func navigateBack() // Pops current destination
}
```

### Navigation Destinations
```swift
enum GameDestination: Hashable {
    case title, game, settings, loadGame
}
```

### Panel State (Internal to TitleScreen)
```swift
enum PanelType {
    case main, settings, loadGame
}
```

---

## Component Integration Points

### Modular Design Philosophy
Each component is completely self-contained:
- TitleBackground: Just gradient, no dependencies
- TitleGridView: Just 3D grid, no dependencies  
- TitleScreen: Combines background + grid + UI, but through composition
- CreatorScreen: Combines background + grid + GameMenuSystem overlay
- GameMenuSystem: Combines GameMenuTop + GameMenuCenter + GameMenuBottom
- No component has knowledge of others' internal state

### Integration Example
```swift
ZStack {
    TitleBackground()    // Gradient layer
    TitleGridView()      // 3D grid layer
    // Custom game UI here
}
```

### Extension Points
- onNewGame: () -> Void callback for game launch -> triggers transition to CreatorScreen
- onCredits: () -> Void callback for credits display
- Panel actions: Print statements ready for implementation
- Settings panels: Placeholder implementations ready for logic

---

## Development Patterns

### Animation Conventions
- Spring animations for smooth transitions
- 0.6 second duration for panel slides
- Asymmetric transitions (right-in, left-out)
- easeInOut timing curves
- Quick screen transitions (0.2s fade + 0.3s black)

### Code Organization
- Platform checks at view level, not component level
- @State for local animation state
- @Observable/@StateObject for shared state
- Conditional compilation for platform differences

### Component Communication
- Callback closures for actions
- No direct component-to-component communication
- Parent views orchestrate component interactions
- Navigation handled at MainGameView level

---

## Known Implementation Details

### TitleScreen Animation Sequence
1. App launches, shows SplashScreen
2. TitleScreen appears with title centered
3. After 2 seconds, title slides up 20 points
4. After 2.4 seconds, buttons slide in from right
5. User interactions trigger panel transitions

### New Game Flow
1. "New Game" button clicked
2. Buttons fade out (0.2s)
3. Black screen transition (0.3s)
4. Navigate to CreatorScreen
5. **CRASH OCCURS** shortly after CreatorScreen loads

### Button Layout Implementation
Uses Group views with conditional content rendering:
```swift
Group {
    if currentPanel == .main {
        TitleButton.primary(title: "New Game", ...)
    } else if currentPanel == .settings {
        TitleButton.secondary(title: "Audio Settings", ...)
    }
}
.transition(trackTransition())
```

All buttons maintain fixed track positions across panel changes.

### RealityKit Camera Setup
```swift
let camera = PerspectiveCamera()
camera.transform.translation = [0, 15, 35]  
camera.look(at: [0, 0, 0], from: camera.transform.translation, relativeTo: nil)
```

---

## Integration with Orchard Ecosystem

### Relationship to Other Components
- **RealityAssets**: Provides file management, GameCore provides UI framework
- **RealitySyntax**: Provides code editing, GameCore provides game menus
- **Game Engine** (future): Will integrate through CreatorScreen GameMenuSystem

### Data Flow
MainGameView -> GameStateManager -> NavigationStack -> TitleScreen -> Panel System
MainGameView -> GameStateManager -> NavigationStack -> CreatorScreen -> GameMenuSystem

### Extension Points for Game Integration
- GameMenuCenter: Primary content area for actual game creation tools
- GameMenuTop: Toolbar for game creation actions
- GameMenuBottom: Status bar and navigation
- GameStateManager: Ready for game state integration
- Save/Load panels: Ready for actual save system integration
- Settings panels: Ready for actual settings implementation

---

## Development Guidelines

### Adding New Panels
1. Add case to PanelType enum in TitleScreen
2. Add content to appropriate track Group
3. Add transition animation
4. Add navigation action if needed

### Adding New Navigation Destinations  
1. Add case to GameDestination enum
2. Add navigationDestination case in MainGameView
3. Add navigation method to GameStateManager

### Platform-Specific Features
Use conditional compilation at view level:
```swift
#if os(macOS)
// macOS-specific implementation
#elseif os(iOS)  
// iOS-specific implementation
#endif
```

### Performance Considerations
- RealityKit grid is performance-optimized with unlit materials
- Lazy loading patterns for complex components
- Minimal state management for smooth animations
- Platform-appropriate animation curves
- **CRITICAL**: Multiple RealityKit instances cause memory leaks and crashes

---

## Common Development Tasks

### Testing Animation System
1. Launch app, verify splash screen
2. Verify title center-to-top animation
3. Test button slide-in timing
4. Test panel transitions (Settings, Continue)
5. Verify track alignment across panels

### Debugging RealityKit Grid
1. Check TitleGridView appears behind UI
2. Verify camera angle shows grid properly
3. Check platform-specific color compilation
4. Test grid performance on target devices
5. **CRITICAL**: Monitor memory usage when transitioning between screens

### Adding Custom Game Content
Replace GameMenuCenter placeholder with actual game content while maintaining:
- MenuButton for navigation back
- Platform-specific navigation hiding
- Consistent visual design
- Windowed overlay approach

---

## Current Limitations

### Critical Issues
- **RealityKit Memory Leaks**: Multiple grid instances cause crashes on macOS
- **Navigation Instability**: Transition from TitleScreen to CreatorScreen problematic
- **Performance Degradation**: App becomes unresponsive after CreatorScreen loads

### Not Yet Implemented
- Actual save/load functionality (UI structure ready)
- Settings persistence (UI structure ready)  
- Credits system (callback ready)
- Game content integration (GameMenuCenter ready)
- Audio system integration
- Input system integration
- **RealityKit memory management solution**

### Placeholder Components
- GameMenuTop: Has basic structure, needs actual toolbar functionality
- GameMenuCenter: Has basic structure, needs actual game creation tools
- GameMenuBottom: Has basic structure, needs actual status/navigation
- GameSettingsView: Has button structure, needs actual settings logic
- GameLoadView: Has slot structure, needs actual save system

### Extension Opportunities
- Theme system for different visual styles
- Additional animation options
- More complex panel layouts
- Custom button styles
- Extended 3D background options
- **Alternative grid implementation to avoid RealityKit issues**

---

*This document reflects the actual current state as of January 2025. The core UI framework is complete but has critical RealityKit memory management issues that cause crashes on macOS when transitioning to CreatorScreen.*