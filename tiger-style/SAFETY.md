# TigerStyle Safety Rules

Safety is the #1 priority. These rules prevent bugs before they happen.

## Assertions (CRITICAL)

### Minimum 2 Assertions Per Function

Every function must have at least 2 assertions. Assertions are:
- Force multiplier for fuzzing
- Documentation of invariants
- Downgrade catastrophic bugs to liveness bugs

```zig
// GOOD: Multiple assertions
fn processMessage(msg: *Message, buffer: []u8) !void {
    assert(msg.len > 0);           // Input validation
    assert(buffer.len >= msg.len); // Capacity check
    // ... process ...
    assert(result.len == msg.len); // Postcondition
}

// BAD: No assertions
fn processMessage(msg: *Message, buffer: []u8) !void {
    // Just processing, no safety checks
}
```

### Pair Assertions

For every property, find TWO code paths to assert it:

```zig
// Assert before write
assert(isValidData(data));
disk.write(data);

// Assert after read (paired)
const loaded = disk.read();
assert(isValidData(loaded));
```

### Split Compound Assertions

```zig
// GOOD: Split for better diagnostics
assert(a);
assert(b);

// BAD: Compound hides which failed
assert(a and b);
```

### Assert Implications

```zig
// Use single-line if for implications
if (is_leader) assert(has_quorum);
```

### Compile-Time Assertions

Assert relationships between constants:

```zig
comptime {
    assert(BUFFER_SIZE >= MAX_MESSAGE_SIZE);
    assert(@sizeOf(Header) == 64);
}
```

## Control Flow

### No Recursion

Recursion makes bounding execution difficult. Use explicit loops:

```zig
// GOOD: Explicit iteration
fn traverse(nodes: []Node) void {
    var stack: [MAX_DEPTH]Node = undefined;
    var depth: usize = 0;
    // ... iterative traversal ...
}

// BAD: Recursion
fn traverse(node: *Node) void {
    for (node.children) |child| {
        traverse(child);  // Unbounded!
    }
}
```

### Split Compound Conditions

```zig
// GOOD: Nested if/else shows all cases
if (a) {
    if (b) {
        // a and b
    } else {
        // a and not b
    }
} else {
    // not a
}

// BAD: Compound condition
if (a and b) {
    // ...
}
```

### State Invariants Positively

```zig
// GOOD: Positive form, natural
if (index < length) {
    // Valid
} else {
    // Invalid
}

// BAD: Negation, harder to reason about
if (index >= length) {
    // Invalid case first
}
```

## Limits on Everything

### Loops Must Have Bounds

```zig
// GOOD: Explicit bound
for (items[0..max_items]) |item| {
    process(item);
}

// GOOD: Assert on event loops
while (true) {
    // Must document this is intentionally unbounded
    assert(is_event_loop);
}
```

### Queues Have Fixed Upper Bounds

```zig
// GOOD: Fixed capacity
var queue: BoundedQueue(Message, 1024) = .{};

// BAD: Unbounded
var queue: ArrayList(Message) = .{};
```

## Memory

### Static Allocation Only

All memory allocated at startup. No dynamic allocation after init:

```zig
// GOOD: Allocate once at init
pub fn init(allocator: Allocator) !Self {
    const buffer = try allocator.alloc(u8, BUFFER_SIZE);
    return .{ .buffer = buffer };
}

// BAD: Dynamic allocation during operation
pub fn process(self: *Self, data: []const u8) !void {
    const temp = try self.allocator.alloc(u8, data.len);  // NO!
    defer self.allocator.free(temp);
}
```

### Never Store Allocator in Struct

Pass allocator as first parameter instead:

```zig
// GOOD: Pass allocator
pub fn deinit(self: *Self, allocator: Allocator) void {
    allocator.free(self.buffer);
}

// BAD: Store allocator
const Self = struct {
    allocator: Allocator,  // NO!
};
```

## Types

### Explicitly-Sized Types

Use `u32`, `u64`, not `usize`:

```zig
// GOOD: Explicit size
count: u32,
offset: u64,

// BAD: Architecture-dependent
count: usize,
```

## Function Size

### Maximum 70 Lines

If a function exceeds 70 lines, split it:

```zig
// GOOD: Small focused functions
fn handleMessage(msg: *Message) !void {
    try validateMessage(msg);
    try processPayload(msg);
    try sendResponse(msg);
}

// Keep control flow in parent, logic in helpers
```

### Push Ifs Up, Fors Down

- Parent functions handle branching
- Helper functions handle iteration/logic
- Keep leaf functions pure

## Error Handling

### All Errors Must Be Handled

Never ignore errors. 92% of catastrophic failures come from unhandled errors:

```zig
// GOOD: Handle or propagate
try operation();
// or
operation() catch |err| {
    log.err("Operation failed: {}", .{err});
    return err;
};

// BAD: Ignore error
_ = operation() catch {};  // Silent failure!
```
