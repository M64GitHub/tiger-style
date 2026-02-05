# TigerStyle Pre-Submit Checklist

Quick validation before committing code.

## Safety (CRITICAL)

- [ ] **Assertions**: Every function has 2+ assertions
- [ ] **No recursion**: All traversals use explicit loops
- [ ] **Bounded loops**: All loops have explicit upper bounds
- [ ] **Static memory**: No dynamic allocation after init
- [ ] **Errors handled**: No ignored errors (no `_ = x catch {}`)
- [ ] **Explicit types**: Using u32/u64, not usize

## Structure

- [ ] **Function size**: Every function < 70 lines
- [ ] **Simple control flow**: No complex compound conditions
- [ ] **Split conditions**: Using nested if/else, not `if (a and b)`
- [ ] **Positive invariants**: `if (valid)` not `if (!invalid)`

## Naming

- [ ] **snake_case**: Functions, variables, files
- [ ] **No abbreviations**: `message` not `msg`
- [ ] **Units last**: `latency_ms_max` not `max_latency`
- [ ] **Acronyms capitalized**: `VSRState` not `VsrState`

## Formatting

- [ ] **Line length**: All lines < 80 characters
- [ ] **zig fmt**: Code is formatted
- [ ] **4-space indent**: Not 2 spaces

## Comments

- [ ] **Why not what**: Comments explain rationale
- [ ] **Sentences**: Comments have capitals and periods
- [ ] **Not obvious**: No comments on self-evident code

## Memory

- [ ] **No stored allocator**: Passed as parameter
- [ ] **No stored Io**: Passed as parameter (Client exception)
- [ ] **In-place init**: Large structs init via pointer

## Quick Severity Guide

| If you find... | Severity |
|----------------|----------|
| Missing assertions | CRITICAL |
| Unbounded loop | CRITICAL |
| Dynamic allocation in hot path | CRITICAL |
| Ignored error | CRITICAL |
| Function > 70 lines | MAJOR |
| usize instead of u32 | MAJOR |
| Stored allocator | MAJOR |
| Wrong naming convention | MINOR |
| Line > 80 chars | MINOR |
| Missing units in name | MINOR |
