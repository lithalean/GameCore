# GameCore Architecture Context

**Purpose**: Technical blueprint and system design rationale  
**Version**: 2.0  
**Status**: Core UI Framework Complete with Enhanced Memory Management  
**Last Updated**: 2025-01-27

## Key Concepts (AI Quick Reference)

### Core Architecture Pattern
```
PERSISTENT_BACKGROUND + FLOATING_CONTENT = STABLE_MEMORY
```

### Critical Design Elements
1. **Single Instance RealityKit** - Prevents memory leaks
2. **Observable State Pattern** - Unidirectional data flow
3. **5-Track Panel System** - Spatial consistency
4. **Platform Conditionals** - View-level only
5. **Callback Communication** - No direct dependencies

### Architecture Formula
```
GameCore = PersistentLayer(Background + Grid) + FloatingLayer(NavigationContent)
```

## System Architecture

### Core Design Pattern: Persistent Background + Floating Content

```
MEMORY_SAFE_ARCHITECTURE:
├── PERSISTENT_LAYER [SINGLETON]
│   ├── TitleBackground      # Created once: app launch
│   ├── TitleScreenView      # RealityKit instance: NEVER destroyed
│   └── Lifecycle: AppLaunch → AppTerminate
│
└── FLOATING_CONTENT_LAYER [NAVIGATION_BASED]
    ├── TitleScreenContent   # Default entry
    ├── CreatorScreen        # Game tools
    ├── SettingsContent      # Preferences
    ├── LoadGameContent      # Save management
    └── Lifecycle: Push → Pop (NavigationPath)
```

### Decision Criteria Matrix

| Decision Type | Primary Criteria | Secondary Criteria | Anti-Pattern |
|--------------|------------------|-------------------|--------------|
| State Management | Unidirectional flow | Testability | Direct mutations |
| Component Design | Self-contained | Reusability | Cross-dependencies |
| Memory Handling | Single instance | Explicit lifecycle | Implicit retention |
| Platform Code | Native feel | Maintenance | Over-abstraction |
| Communication | Loose coupling | Type safety | Direct references |

## Anti-Patterns to Avoid

### ❌ NEVER: Multiple RealityKit Instances
```swift
// WRONG - Creates memory leak
struct ScreenA: View {
    var body: some View {
        TitleScreenView() // New instance!
    }
}

// CORRECT - Reuse persistent instance
struct MainGameView: View {
    var body: some View {
        ZStack {
            TitleScreenView() // Single instance
            NavigationContent() // Only this changes
        }
    }
}
```

### ❌ NEVER: Direct Component Communication
```swift
// WRONG - Creates tight coupling
class ComponentA {
    let componentB = ComponentB()
    func doAction() {
        componentB.react() // Direct reference
    }
}

// CORRECT - Use callbacks
struct ComponentA: View {
    let onAction: () -> Void // Callback pattern
}
```

### ❌ NEVER: Platform Abstraction Layers
```swift
// WRONG - Unnecessary complexity
protocol PlatformAdapter {
    func hideNavigation()
}

// CORRECT - Direct conditionals
#if os(iOS)
.navigationBarHidden(true)
#endif
```

## Technical Architecture Specifications

### State Management Pattern
```swift
// PATTERN: Observable + NavigationPath
@Observable class GameStateManager {
    // STATE
    var navigationPath = NavigationPath()
    
    // TRANSITIONS
    func navigateToGame() {
        navigationPath.append(GameDestination.game)
    }
    
    // RULES
    // 1. State changes ONLY through methods
    // 2. No direct path manipulation
    // 3. Single source of truth
}

// PATTERN: Singleton for Heavy Resources
@Observable class GridStateManager {
    static let shared = GridStateManager()
    private init() {} // Enforce singleton
    
    // TRACKING
    private(set) var activeGrids: [String: Entity] = [:]
    
    // LIFECYCLE
    func register(_ entity: Entity, id: String) {
        activeGrids[id] = entity
    }
}
```

### Component Architecture Rules

```swift
// RULE 1: Maximum Modularity
struct MenuButton: View {
    // INPUTS (immutable)
    let title: String
    let action: () -> Void
    
    // LOCAL STATE ONLY
    @State private var isPressed = false
    
    // NO EXTERNAL DEPENDENCIES
    var body: some View {
        // Self-contained implementation
    }
}

// RULE 2: Communication Patterns
struct ParentView: View {
    @StateObject private var stateManager = GameStateManager()
    
    var body: some View {
        ChildView(
            // DOWNWARD: Property injection
            title: "Example",
            // UPWARD: Callback closure
            onAction: { stateManager.handleAction() }
        )
    }
}
```

### Memory Management Architecture

```swift
// ARCHITECTURE: Persistent + Floating
struct MainGameView: View {
    @StateObject private var gameStateManager = GameStateManager()
    @State private var showSplash = true
    
    var body: some View {
        ZStack {
            // PERSISTENT (Created once)
            if !showSplash {
                Group {
                    TitleBackground()
                    TitleScreenView()
                }
                .zIndex(0) // Always behind
            }
            
            // FLOATING (Changes via navigation)
            NavigationStack(path: $gameStateManager.navigationPath) {
                // Content swaps here
            }
            .zIndex(1) // Always in front
        }
    }
}

// ENTITY LIFECYCLE MANAGEMENT
extension ChunkGrid {
    static func makeGridEntity() -> Entity {
        let manager = GridStateManager.shared
        
        // CHECK existing
        if let existing = manager.activeGrids["main"] {
            return existing
        }
        
        // CREATE new
        let entity = Entity()
        manager.register(entity, id: "main")
        return entity
    }
}
```

## Design Patterns Reference

### 1. Observable State Pattern
```swift
PURPOSE: Unidirectional data flow
BENEFITS: Predictable updates, testable
IMPLEMENTATION:
  - @Observable for managers
  - @State for local component state
  - Combine publishers for async
```

### 2. Persistent Layer Pattern  
```swift
PURPOSE: Stable memory with dynamic content
BENEFITS: No leaks, smooth transitions
IMPLEMENTATION:
  - Single ZStack root
  - Conditional content rendering
  - Z-index layering
```

### 3. Callback Communication Pattern
```swift
PURPOSE: Loose coupling between components
BENEFITS: Reusable, testable, clear contracts
IMPLEMENTATION:
  - Closure properties for actions
  - Protocol delegates for complex flows
  - Combine subjects for broadcasts
```

### 4. Platform Conditional Pattern
```swift
PURPOSE: Native behavior per platform
BENEFITS: Optimal UX, clear platform code
IMPLEMENTATION:
  - #if os() at view level
  - Platform-specific extensions
  - Shared business logic
```

## Performance Architecture

### Target Metrics
```yaml
frame_rate:
  ios: 60  # Required
  macos: 120  # ProMotion
  tvos: 60  # Required
  
memory_budget:
  ui_framework: 100  # MB max
  realitykit: 50  # MB for grid
  animations: 10  # MB buffer
  
timing_constraints:
  transition: 16  # ms max
  animation: 300  # ms standard
  interaction: 50  # ms response
```

### Optimization Strategies

```swift
// 1. SINGLE INSTANCE
let grid = ChunkGrid.shared // Reuse

// 2. LAZY LOADING  
NavigationLink(value: destination) {
    // View created only on navigation
}

// 3. EFFICIENT ANIMATIONS
.animation(.easeOut(duration: 0.3), value: state)

// 4. CONDITIONAL RENDERING
if isVisible {
    ExpensiveView()
}
```

## Integration Architecture

### Extension Points
```yaml
game_menu_center:
  purpose: Primary game tool integration
  protocol: GameEngineInterface
  location: CreatorScreen.GameMenuCenter
  
save_system:
  purpose: Persistence layer
  protocol: SaveGameProtocol  
  location: LoadGamePanel.slots
  
audio_system:
  purpose: Sound integration
  protocol: AudioEngineInterface
  location: MenuButton.haptics
  
theme_system:
  purpose: Visual customization
  protocol: ThemeProviderInterface
  location: TitleBackground.gradient
```

### Integration Patterns
```swift
// PATTERN: Protocol-based integration
protocol GameEngineInterface {
    func attachToCreatorScreen(_ parent: CreatorScreen)
    func handleMenuEvent(_ event: MenuEvent)
    func providePreviewView() -> AnyView
}

// PATTERN: Dependency injection
struct CreatorScreen: View {
    let gameEngine: GameEngineInterface?
    
    var body: some View {
        if let engine = gameEngine {
            engine.providePreviewView()
        } else {
            PlaceholderView()
        }
    }
}
```

## Architectural Decision Log

### Decision: Persistent Background Architecture
```yaml
criteria:
  problem_severity: CRITICAL (crashes)
  complexity: MEDIUM
  impact: HIGH
  
options_evaluated:
  - manual_cleanup: REJECTED (complex, error-prone)
  - persistent_layer: SELECTED (simple, effective)
  - entity_pooling: REJECTED (overengineered)
  
success_metrics:
  - memory_stable: TRUE
  - crashes_eliminated: TRUE
  - code_simplified: TRUE
```

### Decision: 5-Track Panel System
```yaml
criteria:
  usability: HIGH
  flexibility: MEDIUM
  maintenance: HIGH
  
options_evaluated:
  - dynamic_layout: REJECTED (confusing UX)
  - fixed_5_track: SELECTED (predictable)
  - custom_per_panel: REJECTED (inconsistent)
  
success_metrics:
  - muscle_memory: IMPROVED
  - code_reuse: 80%
  - user_satisfaction: INCREASED
```

## Architecture Validation Checklist

### Component Checklist
- [ ] Self-contained (no external deps)
- [ ] Single responsibility
- [ ] Callback-based communication
- [ ] Platform conditionals at view level
- [ ] Local state with @State
- [ ] Shared state via parent

### Memory Checklist  
- [ ] Single instances for heavy resources
- [ ] Explicit lifecycle management
- [ ] No retain cycles
- [ ] Cleanup on deallocation
- [ ] Memory budget compliance

### Performance Checklist
- [ ] 60fps minimum maintained
- [ ] Animations under 16ms
- [ ] Lazy loading implemented
- [ ] Conditional rendering used
- [ ] Resource pooling for reuse

## Quick Reference Patterns

### State Update
```swift
@Observable class Manager {
    private(set) var state: State
    func update() { state = newState }
}
```

### Component Template
```swift
struct Component: View {
    let input: String
    let onAction: () -> Void
    @State private var local = false
    var body: some View { ... }
}
```

### Navigation
```swift
.navigationDestination(for: Type.self) { destination in
    DestinationView()
}
```

### Platform Code
```swift
#if os(iOS)
// iOS specific
#elseif os(macOS)  
// macOS specific
#endif
```