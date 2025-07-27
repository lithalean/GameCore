# GameCore Evolution Log

**Purpose**: Track architectural decisions, their outcomes, and lessons learned  
**Format**: Chronological, with rationale and results

## 2025-01: Memory Architecture Redesign

### The Problem
- **Issue**: Multiple RealityKit instances created during navigation
- **Symptoms**: macOS freezing, memory leaks, performance degradation
- **Root Cause**: Each screen creating its own TitleScreenView with ChunkGrid

### The Decision
- **Choice**: Implement persistent background architecture
- **Alternative Considered**: Manual cleanup of RealityKit entities
- **Rationale**: Single instance pattern prevents accumulation

### The Implementation
```swift
// Before - Each screen had its own background
struct TitleScreen: View {
    var body: some View {
        ZStack {
            TitleBackground()      // Created per screen
            TitleScreenView()      // New RealityKit instance!
            TitleScreenContent()
        }
    }
}

// After - Persistent shared background
struct MainGameView: View {
    var body: some View {
        ZStack {
            // PERSISTENT LAYER
            TitleBackground()      // Created once
            TitleScreenView()      // Single instance
            
            // FLOATING CONTENT
            NavigationContent()    // Only this changes
        }
    }
}
```

### The Results
- **Memory**: Stable at ~85MB (was growing unbounded)
- **Performance**: Consistent 60fps (was degrading over time)
- **Stability**: No more crashes or freezes
- **Platforms**: Fixed issues on all platforms

### Lessons Learned
1. RealityKit entities need explicit lifecycle management
2. Persistent layers can solve complex state problems
3. Single instance patterns work well for heavy resources
4. Architecture changes can be simpler than complex fixes

---

## 2024-12: 5-Track Panel System Design

### The Problem
- **Issue**: Inconsistent button layouts across panels
- **Symptoms**: Confusing navigation, poor spatial memory
- **User Feedback**: "Buttons jump around between screens"

### The Decision
- **Choice**: Fixed 5-track layout with conditional content
- **Alternative**: Dynamic layouts per panel
- **Rationale**: Consistent positioning aids muscle memory

### The Implementation
```swift
// Fixed track positions
Track 1: Primary action (New Game / Audio Settings)
Track 2: Secondary action (Continue / Graphics)  
Track 3: Tertiary action (Settings / Controls)
Track 4: Spacer (visual breathing room)
Track 5: Navigation (Credits / Back)
```

### The Results
- **Usability**: Improved navigation speed
- **Code**: Simpler animation logic
- **Maintenance**: Easy to add new panels
- **Visual**: Consistent, professional appearance

### Lessons Learned
1. Spatial consistency matters more than optimal layout
2. Fixed patterns reduce cognitive load
3. Empty space (Track 4) has value
4. Constraints can improve design

---

## 2024-11: Platform Abstraction Strategy

### The Problem
- **Issue**: Platform-specific code scattered throughout
- **Symptoms**: Difficult to maintain, easy to break
- **Challenge**: iOS, macOS, tvOS have different APIs

### The Decision
- **Choice**: Conditional compilation at view level only
- **Alternative**: Abstract platform layer
- **Rationale**: SwiftUI already provides abstraction

### The Implementation
```swift
// Platform checks at view boundaries
#if os(iOS)
.navigationBarHidden(true)
#elseif os(macOS)
.toolbar(.hidden, for: .windowToolbar)
#elseif os(tvOS)
.navigationBarHidden(true)
#endif
```

### The Results
- **Clarity**: Platform differences visible in one place
- **Performance**: No abstraction overhead
- **Maintenance**: Easy to add platform features
- **Testing**: Can test each platform independently

### Lessons Learned
1. Don't over-abstract platform differences
2. SwiftUI handles most cross-platform concerns
3. Explicit is better than clever
4. Group platform code in clear blocks

---

## 2024-10: Initial Architecture Design

### The Problem
- **Issue**: Starting new game UI framework from scratch
- **Goal**: Modular, reusable components for Orchard ecosystem
- **Requirements**: Must work with future game engines

### The Decision
- **Choice**: Modular component architecture
- **Alternative**: Monolithic game framework
- **Rationale**: Flexibility for unknown future needs

### Key Principles Established
1. **No Direct Dependencies**: Components communicate via callbacks
2. **Self-Contained**: Each component fully functional alone
3. **Platform Agnostic**: Core logic separate from platform code
4. **Extension Ready**: Prepared for future integrations

### The Results
- **Flexibility**: Easy to modify individual components
- **Reusability**: Components used across screens
- **Testing**: Can test components in isolation
- **Integration**: Ready for game engine connection

### Lessons Learned
1. Modular design pays off immediately
2. Callbacks are simple but powerful
3. Planning for extensibility worth effort
4. Start simple, stay simple

---

## Future Decisions to Document

### Pending Architectural Choices
1. **Save System Architecture**
   - Local vs cloud storage
   - Data format (JSON, Core Data, custom)
   - Versioning strategy

2. **Audio Integration**
   - AVFoundation vs third-party
   - Spatial audio for 3D elements
   - Music vs effects separation

3. **Theme System**
   - Dynamic themes vs static
   - User customization depth
   - Performance implications

4. **Game Engine Integration**
   - Communication protocol
   - State synchronization
   - Performance boundaries

---

## Architecture Principles (Refined)

Based on our evolution, these principles have emerged:

1. **Single Instance for Heavy Resources**
   - RealityKit, Audio engines, etc.
   - Explicit lifecycle management

2. **Persistent Layers for Stability**
   - Shared backgrounds reduce complexity
   - Floating content for flexibility

3. **Modular but Not Abstract**
   - Self-contained components
   - Direct platform code when needed

4. **Consistency Over Optimization**
   - Spatial consistency for UX
   - Code patterns for maintenance

5. **Plan for Integration**
   - Clear boundaries
   - Extension points ready
   - Protocol-based interfaces