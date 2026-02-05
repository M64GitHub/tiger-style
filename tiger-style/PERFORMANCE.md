# TigerStyle Performance Rules

Performance is the #2 priority (after safety).

## Design-Time Thinking

### Performance Starts at Design

The biggest wins (1000x) come from design decisions, not micro-optimization:

```
Design phase:     1000x improvements possible
Implementation:   10x improvements possible
Profiling/tuning: 2x improvements typical
```

Think about performance BEFORE writing code.

### Back-of-Envelope Sketches

Before implementing, sketch resource usage:

```
Operation: Process 10k messages/sec
- Network: 10k * 100 bytes = 1 MB/s (well within 1 Gbps)
- Disk: 10k * 100 bytes = 1 MB/s (SSD can do 500 MB/s)
- Memory: 10k * 1KB buffers = 10 MB (fits in L3 cache? No)
- CPU: 10k * 1us = 10ms/s (1% of one core)
```

This reveals bottlenecks before coding.

## Resource Priority

### Optimize Slowest Resources First

Order by latency (compensate for frequency):

| Resource | Typical Latency | Optimize |
|----------|-----------------|----------|
| Network | 1ms - 100ms | First |
| Disk | 100us - 10ms | Second |
| Memory | 100ns - 1us | Third |
| CPU | 1ns - 100ns | Last |

But: A memory cache miss repeated 1000x equals one disk fsync!

## Batching

### Amortize Costs

Batch operations to amortize overhead:

```zig
// GOOD: Batch writes
fn flushBatch(messages: []Message) !void {
    var batch: [BATCH_SIZE][]const u8 = undefined;
    for (messages, 0..) |msg, i| {
        batch[i] = msg.data;
    }
    try disk.writeAll(&batch);  // One syscall
}

// BAD: Individual writes
fn processMessage(msg: Message) !void {
    try disk.write(msg.data);  // Syscall per message!
}
```

### Control Plane vs Data Plane

Separate infrequent control operations from hot data paths:

```zig
// Control plane: Setup, config (can be slow, heavily asserted)
fn configure(options: Options) !void {
    assert(options.validate());
    // ... complex setup ...
}

// Data plane: Hot path (fast, batched, minimal assertions inline)
fn processData(data: []const u8) void {
    // Pre-validated in control plane
    // Minimal work here
}
```

## CPU Predictability

### Let CPU Sprint

Give CPU large chunks of predictable work:

```zig
// GOOD: Predictable loop, no branches
fn sumValues(values: []const u32) u64 {
    var sum: u64 = 0;
    for (values) |v| {
        sum += v;
    }
    return sum;
}

// BAD: Unpredictable branches
fn sumValues(values: []const Value) u64 {
    var sum: u64 = 0;
    for (values) |v| {
        if (v.isValid()) {      // Branch!
            if (v.type == .int) { // Branch!
                sum += v.asInt();
            }
        }
    }
    return sum;
}
```

### Extract Hot Loops

Move hot loops to standalone functions with primitive arguments:

```zig
// GOOD: Standalone function, primitive args
fn scanBuffer(
    buffer: [*]const u8,
    len: usize,
    needle: u8,
) ?usize {
    for (0..len) |i| {
        if (buffer[i] == needle) return i;
    }
    return null;
}

// BAD: Method on struct (compiler must prove field caching)
fn scan(self: *Scanner) ?usize {
    for (0..self.buffer.len) |i| {
        if (self.buffer[i] == self.needle) return i;
    }
    return null;
}
```

## Explicit Over Implicit

### Don't Rely on Compiler Magic

Be explicit about what you want:

```zig
// GOOD: Explicit prefetch options
@prefetch(ptr, .{
    .cache = .data,
    .rw = .read,
    .locality = 3,
});

// BAD: Rely on defaults
@prefetch(ptr, .{});
```

### No Buffer Bleeds

A partially filled buffer with unzeroed padding leaks sensitive data and violates deterministic guarantees:

```zig
// GOOD: Zero the entire buffer, then fill
var buffer: [BUFFER_SIZE]u8 = std.mem.zeroes(
    [BUFFER_SIZE]u8,
);
@memcpy(buffer[0..data.len], data);

// BAD: Unzeroed tail leaks previous contents
var buffer: [BUFFER_SIZE]u8 = undefined;
@memcpy(buffer[0..data.len], data);
// buffer[data.len..] contains garbage!
```

### Explicit Memory Layout

```zig
// GOOD: Explicit layout for cache lines
const Entry = extern struct {
    key: u64,      // 8 bytes
    value: u64,    // 8 bytes
    flags: u32,    // 4 bytes
    _pad: u32,     // 4 bytes (explicit padding)
};
comptime {
    assert(@sizeOf(Entry) == 24);
}
```
