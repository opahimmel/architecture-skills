---
name: semantic-hooks
description: Annotate codebases with machine-readable semantic hooks (@domain, @owns, @routing, @boundary, @extend, @invariant) so AI agents navigate architecture correctly without breaking invariants. Use when user mentions context engineering, agent hooks, semantic annotations, AI navigation hints, wants to annotate code for agents, or when a module lacks architectural metadata.
---

# Semantic Hooks

## Quick start

**File-level header** — top of every architecture-critical module:

```typescript
/**
 * @domain:      canvas-input
 * @owns:        dom-events,hit-testing
 * @depends:     input-actions,state
 * @side-effects:none
 * @invariant:   no-ipc-calls
 */
```

**Inline hooks** — at the seams of the code, with the symbol name repeated so grep finds the marker:

```typescript
// @routing:pointer-down pointerDown — Main entry for all mouse interactions
function pointerDown(event) { ... }

// @boundary:rust-ipc-write source.write — Batched position updates to DB
source.write(shapesSnapshot)

// @extend:shape-types SHAPE_TYPES — Add new shapes here
// @invariant:world-to-screen-math worldToScreen — Do not touch without tests
```

`grep "pointerDown"` now returns both the hook line and the definition — the marker is never invisible.

## Inline hook tags

| Tag | Place it at | Example |
|-----|-------------|---------|
| `@routing:` | Switch statements, central event handlers, message dispatchers | `@routing:keyboard-shortcuts handleKeyDown` |
| `@boundary:` | IPC calls, filesystem access, DOM mutations from non-UI code | `@boundary:fs-read loadImageCache — Loads images into cache` |
| `@extend:` | Union types, arrays, interfaces touched on every new feature of a type | `@extend:color-presets COLOR_PRESETS` |
| `@invariant:` | Math, hit-testing, Z-index logic that silently breaks if "optimized" | `@invariant:snap-math snapToGrid — Do not simplify` |

## File-level header tags

| Tag | Format | Purpose |
|-----|--------|---------|
| `@domain:` | kebab-case | Semantic domain this module owns |
| `@owns:` | comma-separated | Subsystems this module controls |
| `@depends:` | comma-separated | Direct imports that matter architecturally |
| `@side-effects:` | comma-separated or `none` | External state this module touches |
| `@critical-path:` | `true` / `false` | Load-bearing module — extra caution |
| `@invariant:` | kebab-case | Global rule for the entire file |
| `@context:` | kebab-case | Execution context (e.g. `state-mutation-transactions`) |

## Navigation (reading hooks)

Three grep patterns replace file guessing:

```bash
# Find the module that owns a domain area
grep -r "@domain:canvas-input" src/

# Check invariants before writing into a file
grep "@boundary\|@invariant" src/input/pointer-handler.ts

# Find all consumers of a module
grep -r "@depends:grab-module" src/
```

The trigger: when searching for a file, reach for `@domain:` first. When about to modify a module, grep its `@boundary` and `@invariant` before touching anything.

## Workflow: hooking a new module

1. Read the module — identify its architectural role
2. Write the file-level header (domain, owns, depends, side-effects, invariants)
3. Scan for seams: entry points, system boundaries, extension points, fragile logic
4. Place one inline hook per seam — no prose, keywords only
5. Skip anything self-evident from naming alone

## Rules

- **Symbol-mirroring** — inline hooks must repeat the symbol name: `@tag:slug symbolName — description`
- **No prose** — stichwortartig, keine Sätze
- **Sparse** — only at seams an agent would misread without the hint
- **Consistent prefixes** — never invent new ones without updating the guide
- **No duplicates** — one hook per seam; if two tags fit, two separate lines
