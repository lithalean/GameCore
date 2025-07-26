# GameCore — Design Guidelines Document

**Date**: January 2025  
**Target Platform**: iOS 26, macOS 26, tvOS 26  
**Design Language**: Liquid Glass WWDC 2025  
**Development Tools**: Xcode 26, Swift 6.2  
**Purpose**: Technical design specification for AI systems implementing GameCore UI components

---

## Design Philosophy Overview

GameCore implements Apple's Liquid Glass design language introduced at WWDC 2025. This represents the most significant visual update since iOS 7, emphasizing translucency, depth, and fluid responsiveness across all Apple platforms.

### Core Design Principles

**Hierarchy**: Controls float above content as distinct functional layers, creating visual depth while reducing complexity
**Harmony**: Balance between hardware forms, content, and UI controls with rounded elements following natural touch patterns  
**Consistency**: Universal design patterns that adapt across iOS 26, macOS 26, and tvOS 26 while maintaining platform-specific characteristics
**Fluidity**: Dynamic transformations that respond to user interaction, ambient light, and content changes
**Transparency**: Glass-like materials that reflect and refract surrounding content without obscuring functionality

---

## Liquid Glass Material Properties

### Visual Characteristics
- **Translucent material** that reflects and refracts surrounding content
- **Dynamic opacity** that adapts based on content density and environmental factors
- **Real-time rendering** with hardware-accelerated glass effects
- **Contextual adaptation** that changes appearance based on light/dark mode and wallpaper colors
- **Layered depth** with controls floating above content as distinct functional planes

### Technical Implementation
```swift
// Swift 6.2 Liquid Glass APIs
.glassEffect(.light, in: .rect, isEnabled: true)
.glassEffectID("unique-container-id")
.material(.ultraThinMaterial, luminanceReduction: 0.2)
```

### Platform-Specific Adaptations
- **iOS 26**: Full Liquid Glass implementation with motion-responsive translucency
- **macOS 26**: Transparent menu bars and contextual sidebars with workspace reflection
- **tvOS 26**: Depth-aware controls optimized for 10-foot viewing distance

---

## Animation Timing and Motion Design

### Core Animation Values
```swift
// Standard GameCore animation timings
let quickAnimation: Double = 0.2      // Micro-interactions, button presses
let standardAnimation: Double = 0.3   // Panel transitions, state changes  
let slowAnimation: Double = 0.5       // Major layout changes, screen transitions
let extendedAnimation: Double = 0.8   // Title movements, complex transformations
```

### Animation Curves and Springs
```swift
// Liquid Glass spring animations
.animation(.spring(duration: 0.6, bounce: 0.2))  // Panel sliding
.animation(.easeInOut(duration: 0.8))            // Title movement
.animation(.interpolatingSpring(mass: 1.0, stiffness: 100, damping: 10)) // Glass morphing
```

### Transition Specifications
- **Panel slides**: Asymmetric transitions with right-in (trailing edge), left-out (leading edge)
- **Glass morphing**: 0.6 second duration with bounce: 0.2 for organic feel
- **Content focus**: 0.3 second ease-out for tab bar shrinking/expanding
- **Depth changes**: 0.5 second interpolating spring for layer depth adjustments

---

## Spatial Layout and Hierarchy

### 5-Track Layout System
GameCore implements a consistent 5-position layout across all sliding panels:

```
Track 1: Primary action (New Game / Audio Settings / Save Slot 1)
Track 2: Secondary action (Continue / Graphics Settings / Save Slot 2)  
Track 3: Tertiary action (Settings / Controls / Save Slot 3)
Track 4: Spacer (56pt fixed height for visual breathing room)
Track 5: Navigation action (Credits / Back to Menu)
```

### Spacing Constants
```swift
struct GlassConstants {
    // Spacing hierarchy
    static let tight: CGFloat = 4     // Icon-to-text, micro spacing
    static let small: CGFloat = 8     // Button internal padding
    static let medium: CGFloat = 16   // Panel padding, button spacing  
    static let large: CGFloat = 24    // Section spacing, major gaps
    static let xl: CGFloat = 32       // Screen edge padding
    static let xxl: CGFloat = 48      // Major section separation
    
    // Corner radius values  
    static let smallRadius: CGFloat = 6    // Small buttons, badges
    static let mediumRadius: CGFloat = 12  // Standard buttons
    static let largeRadius: CGFloat = 16   // Panels, cards
    static let xlRadius: CGFloat = 20      // Large panels
    static let xxlRadius: CGFloat = 24     // Full-screen overlays
}
```

### Button Sizing and Touch Targets
```swift
// Minimum touch targets (iOS Human Interface Guidelines)
static let minimumTouchTarget: CGFloat = 44  // iOS minimum
static let preferredTouchTarget: CGFloat = 56 // GameCore standard
static let tvOSTouchTarget: CGFloat = 80     // tvOS focus engine

// Button height specifications
static let compactButtonHeight: CGFloat = 44
static let standardButtonHeight: CGFloat = 56  
static let prominentButtonHeight: CGFloat = 64
```

---

## Material and Blur Specifications

### Glass Effect Hierarchy
```swift
// Material thickness levels
enum GlassThickness {
    case ultraThin    // 0.1 opacity - subtle overlay
    case thin         // 0.2 opacity - standard controls  
    case medium       // 0.3 opacity - prominent panels
    case thick        // 0.4 opacity - modal overlays
    case ultraThick   // 0.5 opacity - blocking content
}
```

### Blur Radius Values
```swift
// Gaussian blur specifications for different contexts
static let subtleBlur: CGFloat = 8       // Behind floating controls
static let standardBlur: CGFloat = 12    // Panel backgrounds
static let prominentBlur: CGFloat = 16   // Modal overlays
static let heavyBlur: CGFloat = 24       // Content-blocking modals
```

### Environmental Adaptivity
- **Light mode**: Increased transparency with higher luminance reduction
- **Dark mode**: Deeper opacity with enhanced contrast preservation  
- **High contrast accessibility**: Automatic fallback to solid backgrounds
- **Reduce transparency setting**: Graceful degradation to opaque materials

---

## Color and Visual Effects

### Liquid Glass Color Adaptation
```swift
// Dynamic color system for glass materials
struct LiquidGlassColors {
    // Adapts to surrounding content and wallpaper
    static var adaptiveGlass: Color {
        Color(.systemBackground).opacity(0.7)
    }
    
    // Platform-specific glass tinting
    static var platformGlass: Color {
        #if os(iOS)
        return Color(.systemMaterial)
        #elseif os(macOS)  
        return Color(.underPageBackgroundColor)
        #elseif os(tvOS)
        return Color(.systemFill)
        #endif
    }
}
```

### Reflection and Refraction
- **Content reflection**: UI elements reflect wallpaper and surrounding content with 15% intensity
- **Light interaction**: Controls respond to ambient light changes with subtle luminance shifts
- **Motion parallax**: Slight depth movement (2-4pt) when device orientation changes
- **Color bleeding**: Adjacent content colors subtly influence glass tint with 5% blending

### Depth and Shadow
```swift
// Shadow specifications for floating elements
static let subtleShadow = Shadow(color: .black.opacity(0.1), radius: 4, y: 2)
static let standardShadow = Shadow(color: .black.opacity(0.15), radius: 8, y: 4)  
static let prominentShadow = Shadow(color: .black.opacity(0.2), radius: 12, y: 6)
```

---

## Typography and Readability

### Text Hierarchy in Glass Contexts
```swift
// Typography system optimized for glass backgrounds
static let primaryText: Font = .system(size: 20, weight: .semibold, design: .rounded)
static let secondaryText: Font = .system(size: 16, weight: .medium, design: .default)
static let captionText: Font = .system(size: 12, weight: .regular, design: .default)

// Glass-optimized text colors with enhanced contrast
static let primaryTextOnGlass: Color = .primary.opacity(0.95)
static let secondaryTextOnGlass: Color = .secondary.opacity(0.8)
```

### Contrast Enhancement
- **Text shadows**: Subtle 1pt shadow with 0.3 opacity for text over glass
- **Background blur**: Increased blur radius behind text areas (16pt minimum)
- **Adaptive contrast**: Automatic contrast adjustment based on background luminance
- **Accessibility compliance**: WCAG AA contrast ratios maintained in all transparency levels

---

## Component-Specific Design Rules

### TitleScreen Design Specifications
```swift
// Title animation sequence timing
titleAnimation: {
    delay: 2.0 seconds
    duration: 0.8 seconds  
    curve: .easeOut
    movement: -20pt vertical offset (to maintain screen visibility)
}

// Button slide-in timing
buttonAnimation: {
    delay: 2.4 seconds (0.4s after title starts moving)
    duration: 1.0 seconds
    curve: .easeOut  
    direction: .move(edge: .trailing) + .opacity
}
```

### Panel Transition System Design
```swift
// 5-track sliding panel design pattern
panelTransition: {
    duration: 0.6 seconds
    curve: .easeInOut
    insertion: .move(edge: .trailing) + .opacity
    removal: .move(edge: .leading) + .opacity
    trackMaintenance: true  // Elements stay in fixed positions across panels
}

// Track layout specification for consistent positioning
Track 1: Primary action
Track 2: Secondary action  
Track 3: Tertiary action
Track 4: Fixed spacer (56pt height for visual breathing)
Track 5: Navigation/return action
```

### 3D Grid Background Design
```swift
// Visual specifications for background grid
gridDesign: {
    perspective: action RPG camera angle
    cameraHeight: elevated position for depth perception
    gridPattern: structured with major/minor line hierarchy
    colorScheme: adapts to platform (red borders, white major, gray minor)
    performance: optimized rendering with unlit materials
}
```

---

## Cross-Platform Adaptations

### iOS 26 Specifics
- **Navigation hiding**: `.navigationBarHidden(true)` for full-screen glass experience
- **Haptic feedback**: `.sensoryFeedback(.impact(flexibility: .soft))` for glass interactions
- **Safe area handling**: Glass materials extend to edges with content respecting safe areas
- **Motion responsiveness**: Glass effects respond to device orientation and movement

### macOS 26 Specifics  
- **Window chrome**: `.toolbar(.hidden, for: .windowToolbar)` for seamless glass integration
- **Sidebar transparency**: NavigationSplitView with glass material sidebar
- **Hover effects**: Subtle luminance increase (5%) on mouse hover over glass elements
- **Focus states**: Keyboard navigation with 2pt blue outline on glass controls

### tvOS 26 Specifics
- **Focus engine**: 8pt focus outline with glass-appropriate contrast
- **Larger touch targets**: 80pt minimum for remote navigation
- **Depth perception**: Enhanced shadows and parallax for 10-foot viewing
- **Content protection**: Higher opacity glass to ensure text readability

---

## Performance and Optimization

### Glass Rendering Guidelines
```swift
// Performance-optimized glass implementation
struct OptimizedGlassEffect: ViewModifier {
    let intensity: Double
    let isEnabled: Bool
    
    func body(content: Content) -> some View {
        content
            .background(.ultraThinMaterial, in: .rect(cornerRadius: 12))
            .opacity(isEnabled ? intensity : 1.0)
            .animation(.easeInOut(duration: 0.3), value: isEnabled)
    }
}
```

### Resource Management
- **Conditional rendering**: Glass effects disabled on older hardware (pre-A13 Bionic)
- **Battery optimization**: Reduced glass complexity in low power mode
- **Memory efficiency**: Lazy loading of complex glass materials
- **Thermal management**: Automatic quality reduction under thermal pressure

### Accessibility Considerations
- **Reduce transparency**: Automatic fallback to solid backgrounds when enabled
- **High contrast**: Enhanced contrast ratios with reduced glass opacity
- **Motion sensitivity**: Option to disable glass motion effects
- **Screen reader**: Proper semantic labeling of glass container relationships

---

## Implementation Patterns for AI Systems

### SwiftUI Glass Container Pattern
```swift
struct GlassContainer<Content: View>: View {
    let content: Content
    let glassLevel: GlassThickness
    let cornerRadius: CGFloat
    
    var body: some View {
        content
            .padding(.medium)
            .background(.ultraThinMaterial, in: .rect(cornerRadius: cornerRadius))
            .overlay(
                RoundedRectangle(cornerRadius: cornerRadius)
                    .stroke(.white.opacity(0.2), lineWidth: 1)
            )
            .shadow(color: .black.opacity(0.1), radius: 8, y: 4)
    }
}
```

### Animation State Management Pattern
```swift
// Centralized animation state for consistent timing across components
@Observable class AnimationState {
    var titleOffset: CGFloat = 0
    var panelState: PanelType = .main
    var isTransitioning: Bool = false
    
    func animateToPanel(_ panel: PanelType) {
        guard !isTransitioning else { return }
        withAnimation(.easeInOut(duration: 0.6)) {
            isTransitioning = true
            panelState = panel
        }
        
        // Reset transition state after animation completes
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.6) {
            isTransitioning = false
        }
    }
}
```

---

## Quality Assurance and Testing

### Visual Validation Checklist
- Glass materials maintain readability across all wallpaper types
- Animations complete within specified timing tolerances (±50ms)
- Transparency effects degrade gracefully on older hardware
- All interactive elements meet minimum touch target requirements
- Text contrast ratios exceed WCAG AA standards in all glass contexts

### Cross-Platform Testing Matrix
```
Component          iOS 26    macOS 26    tvOS 26
Glass Materials    ✓ Full    ✓ Full      ✓ Adapted  
Animation Timing   ✓ 60fps   ✓ 120fps    ✓ 60fps
Touch Targets      ✓ 44pt    ✓ Mouse     ✓ 80pt
Accessibility      ✓ Full    ✓ Full      ✓ Full
Performance        ✓ A13+    ✓ M1+       ✓ A12+
```

### Performance Benchmarks
- Glass rendering: <2ms per frame on target hardware
- Animation smoothness: 60fps minimum, 120fps preferred
- Memory usage: <50MB additional for all glass effects
- Battery impact: <5% increase over non-glass implementation

---

## Future Considerations

### iOS 27 Compatibility
Based on Apple's statement that legacy design compatibility will be removed in iOS 27, GameCore's Liquid Glass implementation provides future-proofing for the next major release cycle.

### Hardware Evolution
Design system accommodates rumored "Glasswing" iPhone (2027) with curved glass edges and under-display sensors, ensuring GameCore components will integrate seamlessly with future hardware.

### Accessibility Evolution  
Framework designed to expand with future accessibility APIs while maintaining current WCAG compliance and supporting emerging assistive technologies.

---

*This document reflects Apple's Liquid Glass design language as introduced at WWDC 2025 and implemented in iOS 26, macOS 26, and tvOS 26. All specifications are based on official Apple documentation and developer guidelines current as of January 2025.*