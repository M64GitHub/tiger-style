# tiger-style

A [Claude Code](https://claude.ai/code) skill that brings [TigerBeetle's TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md) coding discipline to your Zig projects. Get real-time guidance, analysis reports, and violation checks for safety-critical code — all from a `/tiger-style` slash command.

## Why

TigerBeetle's engineering style guide is one of the most rigorous approaches to writing reliable systems software. But style guides only work when they're actively applied during development, not reviewed after the fact.

This skill embeds TigerStyle rules directly into your coding workflow. Claude applies them as you write, catches violations before they reach review, and produces structured reports when you need a full audit.

**Priority hierarchy**: Safety > Performance > Developer Experience

## What You Get

- **Writing guidance** — TigerStyle rules applied as you write new Zig code
- **Full analysis reports** — structured breakdown of aligned patterns, violations, and gray areas
- **Quick violation checks** — fast pass to catch problems before commit
- **Severity classification** — CRITICAL / MAJOR / MINOR for prioritized fixes
- **Pre-submit checklist** — built-in validation covering safety, naming, formatting, and memory

## Usage

### Write new code with TigerStyle guidance

```
/tiger-style
```

Loads the full rule set into context. Claude will apply safety, performance, and DX rules as you write code in the current session.

### Analyze an existing file

```
/tiger-style analyze path/to/file.zig
```

Produces a structured report:

```markdown
## TigerStyle Analysis Report

**File**: `path/to/file.zig`

### Summary
| Category   | Count |
|------------|-------|
| Aligned    | 12    |
| Violations | 3     |
| Gray Areas | 1     |

### Aligned (What's Good)
- `process_message():14` — precondition and postcondition assertions present
- `init():5` — static allocation only, no stored allocator

### Violations

#### CRITICAL
| Location | Rule | Issue |
|----------|------|-------|
| `handle_request():42` | Missing assertions | No assertions in function body |

#### MAJOR
| Location | Rule | Issue |
|----------|------|-------|
| `parse_header():89` | Function too long | 87 lines, limit is 70 |

#### MINOR
| Location | Rule | Issue |
|----------|------|-------|
| `server.zig:3` | Naming | camelCase variable name |

### Gray Areas
| Location | Concern | Notes |
|----------|---------|-------|
| `dispatch():55` | Stored Io | Lifetime-scoped and documented — may be justified |

### Recommendations
1. Add precondition and postcondition assertions to `handle_request()`
2. Split `parse_header()` into `parse_header_validate()` and `parse_header_decode()`
```

### Quick violations-only check

```
/tiger-style check
```

Runs a fast scan and reports only violations, skipping aligned patterns and gray areas. Useful as a pre-commit sanity check.

## Installation

Clone or download this repository, then copy the `tiger-style/` skill directory to one of the locations below.

### Project-scoped (shared with your team)

Place the skill inside your project's `.claude/skills/` directory:

```
your-project/
  .claude/
    skills/
      tiger-style/
        SKILL.md
        SAFETY.md
        PERFORMANCE.md
        DX.md
        CHECKLIST.md
```

```bash
# From your project root
mkdir -p .claude/skills
cp -r /path/to/tiger-style/tiger-style .claude/skills/tiger-style
```

Commit `.claude/skills/tiger-style/` to version control so the entire team gets the skill.

### User-scoped (available in all your projects)

Place the skill in your home directory:

```bash
mkdir -p ~/.claude/skills
cp -r /path/to/tiger-style/tiger-style ~/.claude/skills/tiger-style
```

This makes `/tiger-style` available in every project you open with Claude Code.

## Rule Overview

Rules are organized by priority. When rules conflict, higher-priority rules win.

### Safety (Priority #1)

| Rule | Requirement |
|------|-------------|
| Assertions | Minimum 2 per function (preconditions, postconditions, invariants) |
| Pair assertions | Assert the same property from two different code paths |
| No recursion | Use explicit loops with bounded iteration |
| Bounded loops | Every loop must have an explicit upper bound |
| Static memory | All allocation at init; no dynamic allocation during operation |
| Error handling | Every error must be handled; `_ = x catch {}` is forbidden |
| Explicit types | Use `u32`/`u64`, not `usize` |
| Function size | Maximum 70 lines per function |
| Control flow | Split compound conditions into nested if/else |

See [`tiger-style/SAFETY.md`](tiger-style/SAFETY.md) for full rules with code examples.

### Performance (Priority #2)

| Rule | Requirement |
|------|-------------|
| Design-time thinking | Sketch resource usage before writing code |
| Resource priority | Optimize network > disk > memory > CPU |
| Batching | Amortize syscall and I/O overhead |
| CPU predictability | Give the CPU large chunks of branch-free work |
| Hot loop extraction | Standalone functions with primitive arguments |
| Control/data plane | Separate setup paths from hot paths |

See [`tiger-style/PERFORMANCE.md`](tiger-style/PERFORMANCE.md) for full rules with code examples.

### Developer Experience (Priority #3)

| Rule | Requirement |
|------|-------------|
| Naming | `snake_case` for functions, variables, and file names |
| No abbreviations | `message` not `msg`, `connection` not `conn` |
| Units last | `latency_ms_max` not `max_latency` |
| Acronyms | Fully capitalized: `VSRState` not `VsrState` |
| Line length | 80-column hard limit |
| Indentation | 4 spaces |
| Comments | Full sentences explaining *why*, not *what* |
| Struct order | Fields, then types, then methods |
| No stored allocator/Io | Pass as parameters (Client exception for Io) |

See [`tiger-style/DX.md`](tiger-style/DX.md) for full rules with code examples.

## Severity Levels

| Severity | Criteria | Examples |
|----------|----------|----------|
| **CRITICAL** | Safety risk | Missing assertions, unbounded loops, dynamic allocation in hot path, ignored errors |
| **MAJOR** | Maintainability risk | Function over 70 lines, `usize` instead of explicit type, stored allocator |
| **MINOR** | Style deviation | Naming convention, line length, missing units in variable name |

Fix CRITICAL violations immediately. Resolve MAJOR violations before merge. MINOR violations are worth fixing but should not block progress.

## Customizations from TigerStyle

| Rule | TigerBeetle Standard | This Skill |
|------|---------------------|------------|
| Line length | 100 columns | **80 columns** |
| Stored Io | Never | Same, with documented Client exception |

All other rules follow the [original TigerStyle guide](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md).

## File Structure

```
tiger-style/
  SKILL.md           Skill definition: name, modes, report format
  SAFETY.md          Assertions, control flow, memory, error handling
  PERFORMANCE.md     Design-time thinking, batching, CPU predictability
  DX.md              Naming, formatting, ordering, memory patterns
  CHECKLIST.md       Pre-submit checklist with severity guide
```

## Credits

Based on [TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md) by [TigerBeetle](https://github.com/tigerbeetle/tigerbeetle).
