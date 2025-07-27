# GameCore Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the context module structure and efficiently locate information  
**Last Updated**: January 2025

## Quick Reference Matrix

| Information Needed | Primary Module | Secondary Module |
|-------------------|----------------|------------------|
| How something works | ARCHITECTURE.md | IMPLEMENTATION.md |
| Current state/status | IMPLEMENTATION.md | session.json |
| Visual appearance | VISUAL.md | ARCHITECTURE.md |
| User flow/navigation | NAVIGATION.md | IMPLEMENTATION.md |
| External connections | INTEGRATION.yaml | ARCHITECTURE.md |
| Past decisions/why | EVOLUTION.md | ARCHITECTURE.md |
| Recent changes | session.json | EVOLUTION.md |

## Core Context Modules

### ARCHITECTURE.md
**Purpose**: Technical blueprint and system design rationale

**Contains**:
- System architecture patterns (Persistent Background + Floating Content)
- Component hierarchy and relationships
- Memory management strategy (GridStateManager)
- Design patterns (Observable, Modular, Extension Point)
- Integration architecture with Orchard ecosystem
- Performance architecture and constraints
- Core architectural decisions and rationale

**Load for**:
- Understanding WHY something is designed a certain way
- Planning new features that must fit the architecture
- Resolving design conflicts or making architectural choices
- Understanding component relationships and dependencies

---

### IMPLEMENTATION.md
**Purpose**: Current state of the codebase and what actually works

**Contains**:
- Working features with completion status
- Complete file structure (matches actual codebase)
- In-progress features and their status
- Placeholder components awaiting implementation
- Technical debt and cleanup needed
- Actual performance metrics and testing results
- Known issues and limitations

**Load for**:
- Checking what's actually implemented vs planned
- Understanding current file organization
- Finding specific code locations
- Assessing feature completion status
- Planning next implementation steps

---

### VISUAL.md
**Purpose**: Complete visual design system and UI specifications

**Contains**:
- Liquid Glass design language implementation
- Animation timings and transition curves
- Color system and material properties
- Typography hierarchy and text rendering
- Component visual specifications
- Platform-specific visual adaptations
- Accessibility visual modifications
- Touch targets and spacing guidelines

**Load for**:
- Implementing UI components
- Setting animation parameters
- Applying consistent styling
- Ensuring platform-appropriate visuals
- Meeting accessibility requirements

---

### NAVIGATION.md
**Purpose**: Screen flow, user journeys, and state management

**Contains**:
- Complete screen hierarchy
- Navigation destination mappings
- State management patterns (GameStateManager)
- User journey maps for different scenarios
- Transition specifications and timings
- Platform-specific navigation rules
- Keyboard/focus navigation patterns
- Error handling and fallback flows

**Load for**:
- Understanding how users move through the app
- Implementing navigation features
- Managing navigation state
- Planning new screen flows
- Debugging navigation issues

## Dynamic Context Layers

### INTEGRATION.yaml
**Purpose**: External dependencies and connection points

**Contains**:
- Orchard ecosystem integration status
- Apple framework dependencies and versions
- API contracts and protocols
- Data flow patterns
- Platform-specific APIs used
- Future integration considerations
- Testing interfaces and identifiers

**Load for**:
- Integrating with external systems
- Understanding dependencies
- Planning API changes
- Setting up testing
- Checking version requirements

---

### EVOLUTION.md
**Purpose**: Historical record of decisions and their outcomes

**Contains**:
- Chronological architectural decisions
- Problem → Decision → Implementation → Results format
- Lessons learned from each major change
- Code examples showing before/after
- Refined architecture principles
- Pending decisions to be made

**Load for**:
- Understanding why current architecture exists
- Learning from past mistakes/successes
- Making similar architectural decisions
- Onboarding new team members
- Avoiding repeated problems

---

### session.json
**Purpose**: Runtime context for current AI conversation session

**Contains**:
- Current development focus
- Recent changes in this session
- Active warnings to monitor
- Testing priorities
- Discussion history and decisions made
- Performance tracking updates
- Next steps identified

**Load for**:
- Continuing interrupted conversations
- Tracking session-specific decisions
- Monitoring active issues
- Planning immediate next actions
- Maintaining conversation continuity

## Context Loading Strategy

### Task: Bug Fix
1. IMPLEMENTATION.md (current state)
2. session.json (recent changes)
3. ARCHITECTURE.md (design constraints)

### Task: New Feature
1. ARCHITECTURE.md (design patterns)
2. NAVIGATION.md (user flow)
3. VISUAL.md (UI requirements)
4. IMPLEMENTATION.md (integration points)

### Task: Visual Update
1. VISUAL.md (design system)
2. IMPLEMENTATION.md (current components)
3. ARCHITECTURE.md (component patterns)

### Task: Integration Work
1. INTEGRATION.yaml (dependencies)
2. ARCHITECTURE.md (integration points)
3. IMPLEMENTATION.md (current hooks)

### Task: Refactoring
1. EVOLUTION.md (past decisions)
2. ARCHITECTURE.md (patterns)
3. IMPLEMENTATION.md (current state)

## Context Maintenance Protocol

### Update Triggers
- Code Change → Update IMPLEMENTATION.md first
- Design Decision → Update ARCHITECTURE.md + EVOLUTION.md
- Visual Change → Update VISUAL.md
- New Screen/Flow → Update NAVIGATION.md
- External Dependency → Update INTEGRATION.yaml
- Problem Solved → Update EVOLUTION.md

### Validation Requirements
1. No contradictions between modules
2. IMPLEMENTATION.md matches actual code
3. All modules reference consistent versions
4. Evolution entries have complete PDLR format
5. Session context cleared between major tasks

## Module Dependency Graph

```
ARCHITECTURE.md (Why)
    ↓ defines
IMPLEMENTATION.md (What)
    ↓ styled by
VISUAL.md (How it looks)
    ↓ flows through
NAVIGATION.md (User journey)
    ↓ connects via
INTEGRATION.yaml (External links)
    ↓ evolved from
EVOLUTION.md (History)
    ↓ tracked in
session.json (Current state)
```

## Query Resolution Paths

"How does X work?" → ARCHITECTURE.md then IMPLEMENTATION.md  
"What's the current status?" → IMPLEMENTATION.md + session.json  
"How should this look?" → VISUAL.md  
"Where does user go next?" → NAVIGATION.md  
"What external deps?" → INTEGRATION.yaml  
"Why was it built this way?" → EVOLUTION.md  
"What were we just discussing?" → session.json

## AI Processing Instructions

1. Always consult this manifest first to identify required modules
2. Load only modules necessary for the current task
3. Check session.json for ongoing conversation context
4. Update relevant modules immediately when changes occur
5. Cross-reference information across modules for consistency
6. Alert on any contradictions between modules
7. Document significant decisions in EVOLUTION.md

## Module File Paths

```
.context/
├── CONTEXT_MANIFEST.md (this file)
├── core/
│   ├── ARCHITECTURE.md
│   ├── IMPLEMENTATION.md
│   ├── VISUAL.md
│   └── NAVIGATION.md
├── dynamic/
│   ├── INTEGRATION.yaml
│   ├── EVOLUTION.md
│   └── session.json
└── validation/
    └── validate.py
```