# GameCore Architecture Context

**Last Updated**: January 2025  
**Status**: Core UI Framework Complete with Enhanced Memory Management

## System Architecture

### Core Design Pattern: Persistent Background + Floating Content
```
GameCore Architecture
├── PERSISTENT LAYER (Single Instance - Never Destroyed)
│   ├── TitleBackground      # Gradient visual layer
│   └── TitleScreenView      # RealityKit ChunkGrid (3D)
│
└── FLOATING CONTENT LAYER (Navigation-Based)
    ├── TitleScreenContent   # Title screen UI elements
    ├── CreatorScreen        # Game creation interface
    ├── SettingsContent      # Settings panels
    └── LoadGameContent      # Save/load interface
```

### Memory Management Architecture
- **Single Grid Instance**: Only one TitleScreenView/ChunkGrid exists across entire app lifecycle
- **Persistent Background**: Grid and background never get recreated or destroyed
- **Floating Content**: All screens float over persistent background
- **GridStateManager**: Singleton tracks entity lifecycle and prevents memory leaks

### Component Architecture Principles
1. **Maximum Modularity**: Each component is completely self-contained
2. **No Direct Dependencies**: Components communicate through callbacks only
3. **Platform Abstraction**: Conditional compilation at view level, not component level
4. **State Isolation**: Local state with @State, shared state with @Observable

## Technical Architecture

### State Management Pattern
```swift
// Centralized navigation state
@Observable class GameStateManager {
    var navigationPath: NavigationPath = NavigationPath()
    // Push/pop navigation destinations
}

// Memory management for RealityKit
@Observable class GridStateManager {
    static let shared = GridStateManager()
    private(set) var activeGrids: [String: Entity] = [:]
    // Tracks and cleans up entities
}
```

### Cross-Platform Architecture
```swift
// Platform-specific navigation hiding
#if os(iOS)
.navigationBarHidden(true)
#elseif os(macOS)
.toolbar(.hidden, for: .windowToolbar)
#elseif os(tvOS)
.navigationBarHidden(true)
#endif
```

### Component Communication Pattern
- **Callback Closures**: For upward communication (child to parent)
- **Property Injection**: For downward communication (parent to child)
- **Observable State**: For sibling communication (through shared parent)
- **No Direct References**: Components never know about each other

## RealityKit Integration Architecture

### 3D Grid System Design
- **Camera Configuration**: Positioned at [0, 15, 35] for action RPG perspective
- **Grid Specifications**: 100x100 with 1.0 spacing, chunk borders in red
- **Performance Optimizations**: Unlit materials, lazy entity generation
- **Platform Adaptations**: UIColor (iOS) vs NSColor (macOS) handling

### Entity Management Pattern
```swift
// Ensures single instance pattern
ChunkGrid.makeGridEntity() -> Entity {
    // Check GridStateManager for existing instance
    // Create new only if none exists
    // Register with manager for lifecycle tracking
}
```

## Design Patterns Implemented

### 1. Observable State Pattern
- GameStateManager for navigation
- GridStateManager for memory
- AnimationState for UI transitions

### 2. Modular Component Pattern
- Self-contained views with no external dependencies
- Reusable across different contexts
- Platform-agnostic interfaces

### 3. Persistent Layer Pattern
- Background layers created once at app launch
- Content layers swap based on navigation
- Prevents memory accumulation

### 4. Extension Point Pattern
- Callbacks for future game engine integration
- Placeholder components ready for real implementation
- Mock data structures for save/load system

## Performance Architecture

### Target Metrics
- **Frame Rate**: 60fps minimum (iOS/tvOS), 120fps target (macOS)
- **Memory Budget**: <100MB for UI framework
- **Transition Time**: <16ms between screens
- **Entity Count**: ~400 RealityKit entities (stable)

### Optimization Strategies
1. Single RealityKit instance prevents memory leaks
2. Lazy loading for complex components
3. Minimal state management for smooth animations
4. Platform-appropriate animation curves

## Integration Architecture

### Orchard Ecosystem Integration
```
GameCore <-> RealityAssets
         └─> File management and asset pipeline
         
GameCore <-> RealitySyntax  
         └─> Code editing capabilities
         
GameCore <-> Future Game Engine
         └─> Through CreatorScreen.GameMenuCenter
```

### Extension Points
- **GameMenuCenter**: Primary integration point for game tools
- **Save/Load System**: Ready for persistence implementation
- **Settings System**: Structure ready for preferences
- **Audio Integration**: Callbacks prepared for sound system

## Architectural Decisions Log

### Decision: Persistent Background Architecture
**Rationale**: Prevent RealityKit memory leaks and crashes
**Implementation**: Single instance pattern with floating content
**Result**: Eliminated macOS freezing, stable memory usage

### Decision: 5-Track Panel System  
**Rationale**: Consistent spatial layout across all panels
**Implementation**: Fixed positions with conditional content
**Result**: Predictable UI with smooth transitions

### Decision: Platform Conditional Compilation
**Rationale**: Native feel on each platform
**Implementation**: View-level platform checks
**Result**: Optimal experience on iOS, macOS, tvOS

### Decision: Modular Component System
**Rationale**: Enable parallel development and testing
**Implementation**: Self-contained components with callbacks
**Result**: Easy maintenance and extension