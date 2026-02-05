---
name: tiger-style
description: TigerBeetle coding style for safety-critical Zig code. Use for writing new code aligned with TigerStyle, or analyzing existing code to produce structured reports showing aligned patterns, violations, and gray areas.
---

# TigerStyle Skill

Coding style from TigerBeetle optimized for **Safety > Performance > DX**.

## Usage Modes

| Invocation | Mode | Output |
|------------|------|--------|
| `/tiger-style` | Writing | Guidance for writing new code |
| `/tiger-style analyze <file>` | Analysis | Complete report: Aligned + Violations + Gray Areas |
| `/tiger-style check` | Quick check | Violations only (brief) |

## Quick Lookup

| Topic | See | Key Rules |
|-------|-----|-----------|
| Assertions | [SAFETY.md](SAFETY.md) | Min 2 per function, pair assertions |
| Control flow | [SAFETY.md](SAFETY.md) | No recursion, split compound conditions |
| Memory | [SAFETY.md](SAFETY.md) | Static only, no dynamic after init |
| Function size | [SAFETY.md](SAFETY.md) | Max 70 lines |
| Types | [SAFETY.md](SAFETY.md) | Explicit sizes (u32 not usize) |
| Performance | [PERFORMANCE.md](PERFORMANCE.md) | Design-time thinking, batching |
| Resources | [PERFORMANCE.md](PERFORMANCE.md) | Network > disk > memory > CPU |
| Naming | [DX.md](DX.md) | snake_case, units last (latency_ms_max) |
| Formatting | [DX.md](DX.md) | 80 cols, 4-space indent |
| Pre-submit | [CHECKLIST.md](CHECKLIST.md) | Quick validation checklist |

## Our Customizations (vs pure TigerStyle)

| Rule | TigerStyle | Our Standard |
|------|------------|--------------|
| Line length | 100 columns | **80 columns** |
| Assertions | 2+ per function | Same |
| Memory in structs | Never store allocator | Same + never store Io |

## Analysis Report Format

When running `/tiger-style analyze`, produce this structure:

```markdown
## TigerStyle Analysis Report

**File**: `<filename>`
**Date**: <date>

---

### Summary
| Category | Count |
|----------|-------|
| Aligned | N |
| Violations | N |
| Gray Areas | N |

---

### Aligned (What's Good)

Group by category (Assertions, Memory, Naming, Control Flow, etc.):
- `function():line` - What's good about it

---

### Violations

#### CRITICAL
| Location | Rule | Issue |
|----------|------|-------|
| `fn():line` | rule name | description |

#### MAJOR
| Location | Rule | Issue |
|----------|------|-------|

#### MINOR
| Location | Rule | Issue |
|----------|------|-------|

---

### Gray Areas

| Location | Concern | Notes |
|----------|---------|-------|
| `location` | category | Why it's debatable |

---

### Recommendations
1. Actionable fix suggestions
```

## Severity Definitions

| Severity | Criteria |
|----------|----------|
| **CRITICAL** | Safety risk: missing assertions, unbounded loops, dynamic memory |
| **MAJOR** | Maintainability: function too long, complex control flow |
| **MINOR** | Style: naming, formatting, comments |

## Gray Area Examples

Things that might be justified but warrant discussion:
- Storing Io/allocator in struct (if lifetime-scoped and documented)
- Function pointers for callbacks (if event system requires it)
- Generic abstractions (if domain truly requires them)
- Recursion in tree structures (if bounded and proven)
