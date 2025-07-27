# GameCore Visual Design Context

**Design Language**: Liquid Glass WWDC 2025  
**Target Platforms**: iOS 26, macOS 26, tvOS 26  
**Last Updated**: January 2025

## Design System Overview

### Liquid Glass Material Properties
- **Translucent materials** that reflect and refract surrounding content
- **Dynamic opacity** adapting to content density and environment
- **Real-time rendering** with hardware-accelerated glass effects
- **Contextual adaptation** for light/dark modes and wallpaper colors
- **Layered depth** with controls floating above content

### Visual Hierarchy
```
Z-Layer Stack (back to front):
0. TitleBackground     - Gradient base layer
1. TitleScreenView     - 3D ChunkGrid overlay
2. Glass Containers    - Semi-transparent panels
3. Interactive Elements - Buttons and controls
4. Transitions         - Black fade overlays
```

## Color System

### Primary Palette
```swift
// Background gradients
gradientStart: Color.black
gradientEnd: Color.blue.opacity(0.3) / Color.purple.opacity(0.3)

// Glass overlays
gameOverlay: Color.black.opacity(0.8)  // CreatorScreen
panelGlass: Color(.systemMaterial)     // Platform adaptive

// Text on glass
primaryText: Color.primary.opacity(0.95)
secondaryText: Color.secondary.opacity(0.8)
```

### Platform-Specific Colors
- **iOS**: UIColor system colors
- **macOS**: NSColor with desktop adaptations
- **tvOS**: Enhanced contrast for TV viewing

### ChunkGrid Colors
- **Grid Lines**: White (major), Gray 0.3 (minor)
- **Chunk Borders**: Red (#FF0000)
- **Origin Marker**: Yellow
- **Axis Labels**: Red (X), Blue (Z)

## Typography System

### Font Hierarchy
```swift
// Titles and headers
titleFont: .system(size: 28, weight: .bold, design: .rounded)
headerFont: .system(size: 20, weight: .semibold, design: .rounded)

// Body and controls
bodyFont: .system(size: 16, weight: .medium, design: .default)
buttonFont: .system(size: 18, weight: .medium, design: .rounded)
captionFont: .system(size: 12, weight: .regular, design: .default)
```

### Text Rendering on Glass
- 1pt text shadow with 0.3 opacity
- Increased blur behind text areas (16pt minimum)
- Automatic contrast adjustment based on background

## Animation Specifications

### Core Animation Timings
```swift
// Micro-interactions
quickAnimation: 0.2s      // Button presses, fades
standardAnimation: 0.3s   // Panel transitions, state changes
slowAnimation: 0.5s       // Major layout changes
extendedAnimation: 0.8s   // Title movements

// Specific animations
titleSlideDelay: 2.0s
titleSlideDuration: 0.8s
buttonSlideDelay: 2.4s
panelTransitionDuration: 0.6s
```

### Animation Curves
```swift
// Standard curves
.easeInOut(duration: 0.6)    // Panel sliding
.easeOut(duration: 0.8)      // Title movement
.spring(duration: 0.6, bounce: 0.2)  // Glass morphing

// Transition patterns
.asymmetric(
    insertion: .move(edge: .trailing) + .opacity,
    removal: .move(edge: .leading) + .opacity
)
```

### Screen Transition Sequence
1. Button fade out (0.2s easeOut)
2. Black overlay fade in (0.3s easeInOut)
3. Navigation trigger (at 0.5s total)
4. New screen fade in

## Layout System

### 5-Track Layout Specifications
```
Track Heights:
├── Track 1: 56pt (Primary action)
├── Track 2: 56pt (Secondary action)
├── Track 3: 56pt (Tertiary action)
├── Track 4: 56pt (Spacer - visual breathing room)
└── Track 5: 56pt (Navigation/Back)

Total Height: 280pt + spacing
```

### Spacing Guidelines
```swift
struct SpacingConstants {
    static let tight: CGFloat = 4      // Micro spacing
    static let small: CGFloat = 8      // Internal padding
    static let medium: CGFloat = 16    // Component spacing
    static let large: CGFloat = 24     // Section gaps
    static let xl: CGFloat = 32        // Screen padding
    static let xxl: CGFloat = 48       // Major sections
}
```

### Touch Targets
- **iOS Minimum**: 44pt × 44pt
- **GameCore Standard**: 56pt × 56pt  
- **tvOS Focus**: 80pt × 80pt

## Component Visual Specifications

### TitleBackground
```swift
LinearGradient(
    colors: [.black, .blue.opacity(0.3)],
    startPoint: .topLeading,
    endPoint: .bottomTrailing
)
```

### TitleButton Styles
```swift
// Primary (New Game, Start)
.background(Color.blue)
.foregroundColor(.white)

// Secondary (Settings, Options)
.background(Color.gray.opacity(0.3))
.foregroundColor(.primary)

// Standard (Save slots, menu items)
.background(Color.clear)
.overlay(RoundedRectangle(cornerRadius: 8)
    .stroke(Color.gray.opacity(0.5)))
```

### GameMenuSystem (Creator Overlay)
```swift
// Container specifications
maxWidth: 800
maxHeight: 600
background: Color.black.opacity(0.8)
cornerRadius: 12
shadow: Shadow(color: .black.opacity(0.3), radius: 20, y: 10)

// Component heights
menuBarHeight: 60  // Top and bottom bars
```

### ChunkGrid Visual Properties
```swift
// Camera perspective
position: [0, 15, 35]
lookAt: [0, 0, 0]
fov: actionRPG perspective

// Grid rendering
gridSize: 100 × 100
spacing: 1.0 units
lineWidth: 0.05 (minor), 0.1 (major)
materials: unlit for performance
```

## Material & Blur Specifications

### Glass Effect Hierarchy
```swift
enum GlassThickness {
    case ultraThin    // 0.1 opacity - subtle overlay
    case thin         // 0.2 opacity - standard controls
    case medium       // 0.3 opacity - prominent panels
    case thick        // 0.4 opacity - modal overlays
    case gameOverlay  // 0.8 opacity - CreatorScreen
}
```

### Blur Radius Values
- **Subtle**: 8pt - Behind floating controls
- **Standard**: 12pt - Panel backgrounds
- **Prominent**: 16pt - Modal overlays
- **Heavy**: 24pt - Content-blocking
- **Game Overlay**: 20pt - CreatorScreen

### Shadow Specifications
```swift
// Shadow hierarchy
subtleShadow: (color: .black(0.1), radius: 4, y: 2)
standardShadow: (color: .black(0.15), radius: 8, y: 4)
prominentShadow: (color: .black(0.2), radius: 12, y: 6)
overlayShadow: (color: .black(0.3), radius: 20, y: 10)
```

## Platform Visual Adaptations

### iOS 26 Specifics
- Full Liquid Glass implementation
- Motion-responsive translucency
- Haptic feedback on interactions
- Safe area glass extensions

### macOS 26 Specifics
- Transparent window chrome
- 5% luminance increase on hover
- 2pt blue outline for keyboard focus
- Enhanced shadow depth

### tvOS 26 Specifics
- 8pt focus outline with contrast
- Increased opacity for readability
- Enhanced parallax depth
- Larger UI elements (1.5x scale)

## Accessibility Adaptations

### Reduce Transparency Mode
- Glass materials → Solid backgrounds
- Maintain color scheme
- Preserve layout structure
- Enhanced contrast ratios

### High Contrast Mode
- Increased text opacity (1.0)
- Stronger borders (2pt)
- Pure black/white options
- Simplified gradients

### Motion Sensitivity
- Disable glass motion effects
- Instant transitions (no animation)
- Static backgrounds
- Reduced parallax

## Visual State Specifications

### Loading States
- Splash screen: 2s fade
- Content loading: Skeleton screens
- Transition states: Black overlay

### Interactive States
```swift
// Button states
default: baseColor
hover: baseColor + 5% brightness (macOS)
pressed: baseColor - 10% brightness
disabled: baseColor + 50% opacity
focused: 2pt outline (tvOS/keyboard)
```

### Error States
- Red accent color
- Shake animation (0.1s)
- Error icon overlay
- Temporary message (3s)

## Performance Visual Guidelines

### Optimization Priorities
1. Maintain 60fps minimum
2. Reduce overdraw with opaque backgrounds
3. Batch similar visual elements
4. Use Metal rendering when available

### Quality Settings
```swift
enum VisualQuality {
    case low      // No glass, basic shadows
    case medium   // Simple glass, standard shadows  
    case high     // Full glass, all effects
    case ultra    // Enhanced glass, parallax
}
```

### Battery Optimization
- Reduce glass complexity in low power
- Disable non-essential animations
- Lower shadow quality
- Static backgrounds option