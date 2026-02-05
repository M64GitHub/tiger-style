# Analysis Report Format

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
1. Actionable fix suggestions, ordered by severity
```

## Gray Area Examples

Things that might be justified but warrant discussion:
- Storing Io/allocator in struct (if lifetime-scoped and documented)
- Function pointers for callbacks (if event system requires it)
- Generic abstractions (if domain truly requires them)
- Recursion in tree structures (if bounded and proven)
