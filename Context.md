# GameCore — AI Context Document

**Date**: January 2025  
**Status**: Active Development - Core UI Framework Complete with Enhanced Memory Management  
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
│   ├── MenuButton.swift              # Modern menu button with cross-platform haptics
│   └── TitleButton.swift             # Styled game buttons (primary/secondary/standard)
├── Views/
│   ├── MainGameView.swift            # Root navigation with persistent background architecture
│   ├── TitleScreen/
│   │   ├── TitleScreen.swift         # Animated title screen with 5-track sliding panels
│   │   ├── TitleScreenContent.swift  # Modular title screen UI content
│   │   └── TitleScreenView.swift     # Persistent 3D background + ChunkGrid for all screens
│   ├── Navigation/
│   │   └── TitleScreenNavigation.swift # Settings, load game, and other navigation screens
│   ├── EnhancedGameView.swift        # In-game UI wrapper with menu integration
│   ├── CreatorScreen.swift           # Game creator interface with windowed layout
│   └── SplashScreen.swift            # Initial app loading screen
├── Managers/
│   ├── GameStateManager.swift        # Navigation and state management with NavigationPath
│   ├── GridStateManager.swift        # RealityKit memory management and entity tracking
│   └── SlidingPanelManager.swift     # Panel animation system (unused in current implementation)
├── GameCore/Core/
│   └── ChunkGrid.swift               # RealityKit Entity generation for 3D chunk-based grid
├── Styles/
│   ├── GameMenuSystem.swift          # 3-box modular menu system
│   ├── GameMenuTop.swift             # Top menu bar component
│   ├── GameMenuCenter.swift          # Center content area component
│   └── GameMenuBottom.swift          # Bottom menu bar component
└── App/
    └── SplashScreen.swift            # Initial app loading screen
```

### Component Dependencies
- TitleScreen depends on: TitleBackground, TitleScreenView, TitleButton
- MainGameView depends on: TitleScreen, GameStateManager, CreatorScreen, TitleScreenView (persistent)
- TitleScreenView depends on: ChunkGrid (RealityKit Entity), GridStateManager
- CreatorScreen depends on: GameMenuSystem (floats over persistent background)
- GameMenuSystem depends on: GameMenuTop, GameMenuCenter, GameMenuBottom
- All components use cross-platform navigation macros

---

## Current Implementation Status

### Working Components
- **TitleScreen.swift**: Complete 5-track sliding panel system with animations
- **TitleBackground.swift**: Gradient background component (black to blue/purple)
- **TitleScreenView.swift**: Persistent RealityKit 3D ChunkGrid with action RPG camera perspective
- **ChunkGrid.swift**: RealityKit Entity generation with platform-specific colors and chunk borders
- **GameStateManager.swift**: NavigationPath-based state management
- **GridStateManager.swift**: RealityKit memory management and entity cleanup system
- **MainGameView.swift**: Root navigation with persistent background architecture
- **MenuButton.swift**: Cross-platform button with modern sensory feedback
- **CreatorScreen.swift**: Windowed game interface with centered overlay
- **GameMenuSystem.swift**: Modular 3-box layout system
- **Cross-platform navigation**: Conditional compilation for iOS/macOS/tvOS

### Key Features Implemented
- **Persistent Background Architecture**: Single TitleScreenView + ChunkGrid across all screens
- Title animation: slides from center to top (-20 offset) while buttons slide in from right
- 5-track button layout: positions 1,2,3,spacer,5 with consistent alignment across panels
- Sliding panel system: main/settings/loadGame panels with asymmetric transitions
- RealityKit integration: camera at [0,15,35] looking at [0,0,0] for action RPG perspective
- Cross-platform navigation: iOS navigationBarHidden, macOS toolbar hidden, tvOS navigationBarHidden
- Screen transitions: Fade to black transition between title and creator screens
- Windowed interface: GameMenuSystem overlays on background with constrained sizing
- **Memory Management**: GridStateManager tracks entities and prevents multiple grid instances

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
// NEW: Persistent background architecture
ZStack {
    // PERSISTENT LAYER (never changes)
    TitleBackground()    // Gradient component
    TitleScreenView()    // ChunkGrid component with memory management
    
    // FLOATING CONTENT LAYER (changes per screen)
    TitleScreenContent() // or CreatorScreen() or SettingsContent()
}

// CreatorScreen now just floating content
GameMenuSystem()     // Centered overlay (800x600)
    .background(Color.black.opacity(0.8))
    .cornerRadius(12)
    .shadow(...)
```

---

## 3D Grid Implementation

### RealityKit Integration
- Uses RealityView with PerspectiveCamera
- Camera positioned at [0, 15, 35] for action RPG angle
- Grid generated via ChunkGrid.makeGridEntity()
- Platform-specific color handling (UIColor/NSColor)
- **Enhanced Memory Management**: GridStateManager tracks entity lifecycle

### Grid Specifications
- 100x100 grid with 1.0 spacing
- Red chunk borders at edges (hence "ChunkGrid")
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
- **Entity tracking and cleanup via GridStateManager**

---

## **MEMORY MANAGEMENT SOLUTION - IMPLEMENTED**

### **Fixed Architecture**
- **Single Grid Instance**: Only one TitleScreenView/ChunkGrid exists across entire app
- **Persistent Background**: Grid and background never get recreated or destroyed
- **Floating Content**: All screens (CreatorScreen, Settings, etc.) float over persistent background
- **GridStateManager**: Tracks entity count and prevents memory leaks

### **Technical Implementation**
```swift
// NEW: Persistent background in MainGameView
ZStack {
    // PERSISTENT BACKGROUND + GRID (NEVER CHANGES)
    if !showSplash {
        TitleBackground()
        TitleScreenView()  // Single grid instance
    }
    
    // FLOATING CONTENT LAYER
    switch currentDisplayState {
        case .titleScreen: TitleScreenContent()
        case .transition: ScreenTransitionOverlay()
        // etc.
    }
}

// GridStateManager prevents multiple instances
@Observable class GridStateManager {
    static let shared = GridStateManager()
    private(set) var activeGrids: [String: Entity] = [:]
    // Tracks and cleans up entities
}
```

### **Problem Resolution**
- ✅ **macOS Freeze**: Fixed by eliminating multiple grid instances
- ✅ **Memory Leak**: Prevented by persistent architecture + GridStateManager
- ✅ **Performance Degradation**: Eliminated by single grid approach
- ✅ **Navigation Issues**: Resolved by floating content over persistent background

### **Previous Critical Issues (NOW RESOLVED)**
- ❌ ~~macOS Freeze: App locks up and requires force quit shortly after CreatorScreen loads~~
- ❌ ~~Memory Leak: RealityKit entities not properly deallocated when views switch~~
- ❌ ~~Performance Degradation: Multiple grid instances accumulate in memory~~
- ❌ ~~Navigation Issues: Sometimes appears to go back to splash screen instead of forward~~

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

### GridStateManager (NEW)
```swift
@Observable class GridStateManager {
    static let shared = GridStateManager()
    private(set) var activeGrids: [String: Entity] = [:]
    private(set) var gridCount: Int = 0
    
    func registerGrid(_ entity: Entity, identifier: String)
    func cleanupGrid(identifier: String)
    func printDebugInfo() // For development monitoring
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
- TitleScreenView: ChunkGrid + camera, with GridStateManager integration
- TitleScreenContent: Modular UI content that floats over background
- CreatorScreen: Just GameMenuSystem overlay, floats over persistent background
- GameMenuSystem: Combines GameMenuTop + GameMenuCenter + GameMenuBottom
- No component has knowledge of others' internal state

### Integration Example
```swift
// NEW: Persistent + Floating architecture
ZStack {
    TitleBackground()     // Persistent gradient layer
    TitleScreenView()     // Persistent 3D grid layer
    // Any floating content here
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

### New Game Flow (UPDATED - NO MORE CRASH)
1. "New Game" button clicked
2. Buttons fade out (0.2s)
3. Black screen transition (0.3s)
4. Navigate to CreatorScreen (floats over persistent background)
5. ✅ **NO CRASH** - persistent architecture prevents memory issues

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
MainGameView -> GameStateManager -> NavigationStack -> CreatorScreen -> GameMenuSystem (floating)

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
4. Create floating content (no background/grid needed)

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
- ChunkGrid is performance-optimized with unlit materials
- Lazy loading patterns for complex components
- Minimal state management for smooth animations
- Platform-appropriate animation curves
- **RESOLVED**: Memory management via persistent architecture

---

## Common Development Tasks

### Testing Animation System
1. Launch app, verify splash screen
2. Verify title center-to-top animation
3. Test button slide-in timing
4. Test panel transitions (Settings, Continue)
5. Verify track alignment across panels

### Debugging RealityKit Grid
1. Check TitleScreenView appears behind UI
2. Verify camera angle shows grid properly
3. Check platform-specific color compilation
4. Test grid performance on target devices
5. **RESOLVED**: Memory usage now stable with persistent architecture

### Adding Custom Game Content
Replace GameMenuCenter placeholder with actual game content while maintaining:
- MenuButton for navigation back
- Platform-specific navigation hiding
- Consistent visual design
- Windowed overlay approach
- Floating over persistent background

---

## Current Status

### Resolved Issues
- ✅ **RealityKit Memory Leaks**: Fixed with persistent architecture
- ✅ **Navigation Stability**: Resolved with floating content approach
- ✅ **Performance**: Stable with single grid instance
- ✅ **macOS Compatibility**: Grid loads and maintains properly

### Working Features
- Persistent 3D ChunkGrid background across all screens
- Smooth navigation transitions without memory issues
- Modular floating content architecture
- Cross-platform stability (iOS, macOS, tvOS)
- Enhanced memory monitoring via GridStateManager

### Not Yet Implemented
- Actual save/load functionality (UI structure ready)
- Settings persistence (UI structure ready)  
- Credits system (callback ready)
- Game content integration (GameMenuCenter ready)
- Audio system integration
- Input system integration

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
- Enhanced GridStateManager monitoring

---

*This document reflects the current state as of January 2025. The core UI framework is complete with resolved RealityKit memory management through persistent background architecture. The framework is stable and ready for game content integration.*