# TigerStyle Developer Experience Rules

DX is the #3 priority (after safety and performance).

## Naming

### Get Nouns and Verbs Right

Names capture the essence. Take time to find the perfect name:

```zig
// GOOD: Clear mental model
const Connection = struct { ... };
fn connect() !Connection { ... }
fn disconnect(conn: *Connection) void { ... }

// BAD: Vague or misleading
const Handler = struct { ... };  // Handler of what?
fn process() !void { ... }       // Process what?
```

### snake_case Everywhere

Functions, variables, file names:

```zig
// GOOD
fn read_message() !Message { ... }
var message_count: u32 = 0;
// File: message_parser.zig

// BAD
fn readMessage() !Message { ... }  // camelCase
var messageCount: u32 = 0;
// File: MessageParser.zig
```

### No Abbreviations

Except for primitive loop indices:

```zig
// GOOD
var message: Message = undefined;
var connection: Connection = undefined;
for (0..count) |i| { ... }  // Loop index OK

// BAD
var msg: Message = undefined;    // Abbreviated
var conn: Connection = undefined;
```

### Acronyms Properly Capitalized

```zig
// GOOD
const VSRState = struct { ... };
const HTTPClient = struct { ... };

// BAD
const VsrState = struct { ... };
const HttpClient = struct { ... };
```

### Units and Qualifiers Last

Sort by descending significance:

```zig
// GOOD: Most significant first, units last
latency_ms_max: u64,
latency_ms_min: u64,
buffer_size_bytes: u32,
timeout_seconds: u32,

// BAD: Units first or missing
max_latency_ms: u64,  // Doesn't line up with min
latency: u64,         // Units missing!
```

### Same-Length Names for Related Variables

```zig
// GOOD: Same length, lines up in code
source: []const u8,
target: []u8,
source_offset: usize,
target_offset: usize,

// BAD: Different lengths
src: []const u8,
dest: []u8,
src_offset: usize,
dest_offset: usize,
```

### Prefix Helpers with Caller Name

```zig
// GOOD: Shows call relationship
fn read_sector() !void {
    // ...
    try read_sector_validate();
    try read_sector_process();
}

fn read_sector_validate() !void { ... }
fn read_sector_process() !void { ... }
```

### Callbacks Go Last

```zig
// GOOD: Callback last (mirrors invocation order)
fn subscribe(
    subject: []const u8,
    options: Options,
    callback: *const fn(*Message) void,
) !void { ... }
```

## Ordering

### Important Things Near Top

File read top-down. Put main first:

```zig
// File structure
pub fn main() !void { ... }      // Entry point first

pub fn init() !Self { ... }       // Then public API

fn helper() void { ... }          // Then helpers
```

### Struct Order: Fields, Types, Methods

```zig
const Client = struct {
    // 1. Fields first
    state: State,
    connection: Connection,

    // 2. Types second
    const State = enum { disconnected, connected };
    const Self = @This();

    // 3. Methods last
    pub fn init() !Self { ... }
    pub fn connect() !void { ... }
};
```

## Formatting

### 80-Column Hard Limit

Our standard (stricter than TigerStyle's 100):

```zig
// GOOD: Fits in 80 columns
fn process_message(message: *Message, options: Options) !void {
    // ...
}

// If too long, use trailing comma for auto-wrap:
fn process_message(
    message: *Message,
    options: Options,
    callback: *const fn() void,
) !void {
    // ...
}
```

### 4-Space Indentation

More visible than 2 spaces.

### Comments Are Sentences

```zig
// GOOD: Proper sentence, explains why
// We buffer messages to amortize syscall overhead.
var buffer: [BATCH_SIZE]Message = undefined;

// BAD: Fragment, states the obvious
// buffer for messages
var buffer: [BATCH_SIZE]Message = undefined;
```

### Comments Above, Not Inline (if long)

```zig
// GOOD: Comment above if it doesn't fit
// This offset accounts for the header size which varies by version.
const offset = base_offset + header_size;

// BAD: Long inline comment
const offset = base_offset + header_size; // accounts for header...
```

## Memory Patterns

### Never Store Allocator or Io

Pass as parameters:

```zig
// GOOD
pub fn deinit(self: *Self, allocator: Allocator) void { ... }
pub fn read(self: *Self, io: Io) !Message { ... }

// BAD
const Self = struct {
    allocator: Allocator,  // NO!
    io: Io,                // NO! (except documented exceptions)
};
```

### In-Place Initialization for Large Structs

```zig
// GOOD: In-place, no copy
pub fn init(target: *LargeStruct) !void {
    target.* = .{
        .field = value,
    };
}

var instance: LargeStruct = undefined;
try instance.init();

// BAD: Returns copy
pub fn init() !LargeStruct {
    return .{ .field = value };  // Copied on return
}
```

### Variables Close to Use

Don't declare early, don't leave around late:

```zig
// GOOD: Declared where needed
fn process() !void {
    // ... other work ...

    const result = compute();  // Declared here
    try validate(result);      // Used immediately
}

// BAD: Declared too early
fn process() !void {
    const result = compute();  // Declared early

    // ... 50 lines of other work ...

    try validate(result);      // Used much later
}
```

## Dependencies

### Zero Dependencies

Apart from Zig toolchain. Dependencies bring:
- Supply chain risk
- Safety/performance unknowns
- Slow install times

### Zig for Everything

Scripts too:

```zig
// GOOD: scripts/build_docs.zig
// Cross-platform, type-safe

// BAD: scripts/build_docs.sh
// Platform-specific, fragile
```
