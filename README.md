# zlap

Lightweight microbenchmarking with warmup, automatic batching, and allocation tracking.

## Installation

```sh
zig fetch --save=zlap git+https://github.com/lovez-dev/zlap.git#main
```

Add the module to your executable in `build.zig`:

```zig
const zlap = b.dependency("zlap", .{
    .target = target,
    .optimize = optimize,
});
exe.root_module.addImport("zlap", zlap.module("zlap"));
```

## Usage

```zig
const std = @import("std");
const zlap = @import("zlap");

pub fn main(init: std.process.Init) !void {
    const result = try zlap.runConfigured(init.io, .{
        .batch_size = 100,
        .batch_repeats = 10,
        .sample_count = 30,
        .rounds = 3,
    }, allocAndFill);
    result.print("alloc + memset + free 4KB");
}

fn allocAndFill(allocator: std.mem.Allocator, operations: usize) !void {
    for (0..operations) |_| {
        const buffer = try allocator.alloc(u8, 4096);
        @memset(buffer, 0xAB);
        std.mem.doNotOptimizeAway(buffer);
        allocator.free(buffer);
    }
}
```

When working from a checkout of zlap, run the included example with:

```sh
zig build bench -Doptimize=ReleaseFast
```

The workload must complete the requested number of operations and release its resources before returning. Setup and cleanup inside the workload are timed too. See the [API guide](docs/api.md) for configuration, allocation tracking, and result interpretation.

## License

[MIT](LICENSE)
