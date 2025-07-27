# GameCore Visual Design Context

**Purpose**: Complete visual design system and UI specifications  
**Version**: 2.0  
**Design Language**: Liquid Glass WWDC 2025  
**Last Updated**: 2025-01-27

## Visual Hierarchy Diagram

```
Z-LAYERS (back→front):
├─[0] TitleBackground      █░░░░░░░░░ (Gradient base)
├─[1] TitleScreenView      ░█░░░░░░░░ (3D ChunkGrid)
├─[2] Glass Containers     ░░█░░░░░░░ (Panels/overlays)
├─[3] Interactive Elements ░░░█░░░░░░ (Buttons/controls)
└─[4] Transitions         ░░░░█░░░░░ (Black overlays)

VISUAL_WEIGHT:
Background: 10% opacity
Grid: 30% opacity  
Glass: 40-80% opacity
Controls: 95% opacity
Overlays: 80% opacity
```

## Quick Reference Tables

### Color Palette
| Element | Light Mode | Dark Mode | Opacity | Hex |
|---------|------------|-----------|---------|-----|
| **Background Start** | Black | Black | 1.0 | #000000 |
| **Background End** | Blue | Purple | 0.3 | #0000FF / #800080 |
| **Game Overlay** | Black | Black | 0.8 | #000000 |
| **Grid Lines** | White | White | 1.0 | #FFFFFF |
| **Grid Minor** | Gray | Gray | 0.3 | #808080 |
| **Chunk Borders** | Red | Red | 1.0 | #FF0000 |
| **Text Primary** | Primary | Primary | 0.95 | System |
| **Text Secondary** | Secondary | Secondary | 0.8 | System |

### Spacing System
| Token | Value | Usage | Touch Safe |
|-------|-------|-------|------------|
| **tight** | 4pt | Micro spacing | ❌ |
| **small** | 8pt | Internal padding | ❌ |
| **medium** | 16pt | Component spacing | ❌ |
| **large** | 24pt | Section gaps | ❌ |
| **xl** | 32pt | Screen padding | ✅ |
| **xxl** | 48pt | Major sections | ✅ |
| **touch** | 44pt | Min touch target | ✅ |
| **button** | 56pt | Standard button | ✅ |
| **tv** | 80pt | tvOS focus target | ✅ |

### Typography Scale
| Style | Size | Weight | Design | Line Height |
|-------|------|--------|--------|-------------|
| **Title** | 28pt | Bold | Rounded | 34pt |
| **Header** | 20pt | Semibold | Rounded | 24pt |
| **Body** | 16pt | Medium | Default | 20pt |
| **Button** | 18pt | Medium | Rounded | 22pt |
| **Caption** | 12pt | Regular | Default | 16pt |

## Platform Difference Matrix

| Feature | iOS | macOS | tvOS |
|---------|-----|-------|------|
| **Nav Bar** | Hidden | Hidden toolbar | Hidden |
| **Touch Target** | 44pt min | 44pt min | 80pt min |
| **Hover Effects** | ❌ | ✅ +5% brightness | ❌ |
| **Focus Effects** | ❌ | 2pt blue outline | 8pt white glow |
| **Haptics** | ✅ Selection | ❌ | ❌ |
| **Safe Areas** | ✅ Required | ❌ | ✅ TV safe |
| **Shadows** | Standard | Enhanced depth | Reduced |
| **Blur Radius** | 12pt | 16pt | 8pt |
| **Animation Boost** | 1.0x | 1.0x | 1.5x |

## Animation Timing Reference

```yaml
# Core Timings
INSTANT: 0
QUICK: 200ms
STANDARD: 300ms  
SMOOTH: 500ms
EXTENDED: 800ms

# Specific Animations
title_slide:
  delay: 2000ms
  duration: 800ms
  curve: easeOut
  
button_reveal:
  delay: 2400ms
  duration: 600ms
  curve: easeIn
  
panel_transition:
  duration: 600ms
  curve: easeInOut
  asymmetric: true
  
screen_fade:
  out: 200ms
  black: 300ms
  total: 500ms
```

## Glass Material Specifications

### Material Hierarchy
```yaml
ULTRA_THIN:
  opacity: 0.1
  blur: 4pt
  use: "Subtle overlays"
  
THIN:
  opacity: 0.2
  blur: 8pt
  use: "Standard controls"
  
MEDIUM:
  opacity: 0.3
  blur: 12pt
  use: "Prominent panels"
  
THICK:
  opacity: 0.4
  blur: 16pt
  use: "Modal overlays"
  
GAME_OVERLAY:
  opacity: 0.8
  blur: 20pt
  use: "CreatorScreen"
```

### Platform Glass Rendering
```swift
// iOS Implementation
.background(.ultraThinMaterial)
.background(Color.black.opacity(0.3))

// macOS Implementation  
.background(.underPageBackgroundMaterial)
.background(Color.black.opacity(0.35))

// tvOS Implementation
.background(Color.black.opacity(0.4))
.blur(radius: 8)
```

## Component Visual Specifications

### TitleBackground
```yaml
type: LinearGradient
colors: [black, blue.opacity(0.3)]
startPoint: topLeading
endPoint: bottomTrailing
animation: none
platform_variants: none
```

### ChunkGrid Visual
```yaml
camera:
  position: [0, 15, 35]
  target: [0, 0, 0]
  fov: actionRPG
  
grid:
  size: 100x100
  spacing: 1.0
  line_width:
    minor: 0.05
    major: 0.1
  colors:
    grid: white
    borders: red (#FF0000)
    origin: yellow
    x_axis: red
    z_axis: blue
    
materials:
  type: unlit
  shadows: false
  reflections: false
```

### Button Styles
```yaml
primary:  # New Game, Start
  background: blue
  foreground: white
  cornerRadius: 8
  padding: [16, 32]
  shadow: standard
  
secondary:  # Settings, Options
  background: gray.opacity(0.3)
  foreground: primary
  cornerRadius: 8
  padding: [16, 32]
  shadow: subtle
  
standard:  # Menu items
  background: clear
  foreground: primary
  border: gray.opacity(0.5)
  borderWidth: 1
  cornerRadius: 8
  padding: [12, 24]
```

### 5-Track Layout
```yaml
track_height: 56pt
track_spacing: 8pt
total_height: 320pt

tracks:
  1: "Primary action"
  2: "Secondary action"
  3: "Tertiary action"
  4: "Spacer (empty)"
  5: "Navigation/Back"
  
alignment: center
container_padding: 32pt
```

## Shadow Specifications

| Type | Color | Opacity | Radius | X | Y |
|------|-------|---------|--------|---|---|
| **Subtle** | Black | 0.1 | 4pt | 0 | 2pt |
| **Standard** | Black | 0.15 | 8pt | 0 | 4pt |
| **Prominent** | Black | 0.2 | 12pt | 0 | 6pt |
| **Overlay** | Black | 0.3 | 20pt | 0 | 10pt |

## State Visual Specifications

### Interactive States
```yaml
default:
  opacity: 1.0
  scale: 1.0
  brightness: 0
  
hover:  # macOS only
  opacity: 1.0
  scale: 1.0
  brightness: +5%
  
pressed:
  opacity: 0.9
  scale: 0.98
  brightness: -10%
  
disabled:
  opacity: 0.5
  scale: 1.0
  brightness: 0
  
focused:  # tvOS/keyboard
  opacity: 1.0
  scale: 1.05
  outline: 2pt
```

### Loading States
```yaml
splash:
  duration: 2000ms
  fade_curve: easeOut
  background: black
  
skeleton:
  animation: pulse
  duration: 1500ms
  opacity: [0.3, 0.6]
  
transition:
  overlay: black
  opacity: 0.8
  duration: 300ms
```

## Accessibility Adaptations

### Reduce Transparency
```yaml
changes:
  glass_materials: → solid_backgrounds
  blur_effects: → removed
  shadows: → reduced_50%
  overlays: → increased_opacity
  
maintains:
  - layout_structure
  - color_scheme
  - animations
  - typography
```

### High Contrast
```yaml
changes:
  text_opacity: → 1.0
  borders: → 2pt_width
  colors: → pure_black_white
  gradients: → solid_colors
  
ratios:
  text_on_background: 7:1
  button_borders: 4.5:1
  focus_indicators: 3:1
```

### Reduce Motion
```yaml
disabled:
  - glass_motion_effects
  - parallax_depth
  - spring_animations
  - extended_transitions
  
replaced_with:
  - instant_transitions
  - static_backgrounds
  - simple_fades
  - reduced_durations
```

## Performance Visual Settings

### Quality Presets
```yaml
LOW:
  glass: false
  shadows: basic
  blur: false
  animations: reduced
  target_fps: 30
  
MEDIUM:
  glass: simple
  shadows: standard
  blur: 8pt_max
  animations: standard
  target_fps: 60
  
HIGH:
  glass: full
  shadows: enhanced
  blur: unlimited
  animations: full
  target_fps: 60
  
ULTRA:
  glass: enhanced
  shadows: raytraced
  blur: enhanced
  animations: physics
  target_fps: 120
```

### Battery Optimization
```yaml
low_power_mode:
  reduce_glass: 50%
  disable_animations: non_essential
  lower_shadow_quality: true
  static_backgrounds: true
  reduce_fps: 30
```

## Visual Debug Overlay

```yaml
debug_info:
  show_fps: true
  show_memory: true
  show_touches: true
  show_focus: true
  show_safe_areas: true
  show_grid_lines: true
  
colors:
  fps_good: green
  fps_warning: yellow
  fps_bad: red
  touch_point: blue
  focus_ring: purple
  safe_area: orange
```

## Quick Visual Formulas

```
GLASS_OPACITY = base_opacity + (blur_radius / 100)
SHADOW_SPREAD = radius * 0.5 + y_offset
ANIMATION_FEEL = duration * curve_intensity
TOUCH_TARGET = max(content_size, platform_minimum)
CONTRAST_RATIO = (L1 + 0.05) / (L2 + 0.05)
```