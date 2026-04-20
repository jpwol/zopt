## zopt

An extremely minimal argument parsing library for Zig

---

**zopt** is a cli argument parsing library designed to be as minimal and user friendly as possible.

On the command line, it functions similarly to `getopt`. However, in code it runs on a _struct_ config based approach.

### Requirements

- [Zig](https://ziglang.org) 0.16.0

### Usage

In code, the library is used like this

```zig
const std = @import("std");
const zopt = @import("zopt");

const Opts = struct {
    column: bool = false,
    count: u64 = 0,
    input: []const u8 = "",

    pub const shorthands = .{
        .C = "column",
        .c = "count",
        .i = "input",
    };
};

pub fn main(init: std.process.Init) !void {
    const allocator = init.arena.allocator();
    const res = try zopt.parse(Args, allocator, init.minimal.args);

    std.debug.print("column: {}\ncount: {d}\n", .{res.flags.column, res.flags.count});

    std.debug.print("positionals: ", .{});
    for (res.argv) |arg| {
        std.debug.print("{s}, ", .{arg});
    }

    std.debug.print("\n", .{});
}
```

where the struct `Opts` (which could be any name) defines the command line options and their types. You can (and should) give the fields
default values in the case of the user not defining them on the command line.

The struct fields are the long names that will be used on the command line, where in our example, the long options would be `--column` and `--count`.

`shorthands` defines the short (and more commonly used) versions of these flags. Again from the example, these would be `-C` and `-c`.

`parse()` will parse the command line into two fields, `flags` and `argv`.

- `flags` is an instance of your struct with the fields filled by the parsed command line, or the defaults.
  As seen, this instance can be accessed through `res.flags`.
- `argv` is the rest of the command line. This includes the executable name at `argv[0]` and then all the following positionals.

#### On the command line

Still following our example, these flags can be used as such

- `./prog -C` or `./prog --column` will enable the boolean `column` flag defined in the struct
- `./prog -c 5` or `./prog --count=5` or `./prog --count 5` will set the value of `count` to 5.
  Note that using the flag but not providing a value with throw an error.
- `./prog --column --count=5` or `./prog -C --count=5` uses both flags.
  - If using the short versions, they can be chained on one `-`: `./prog -Cc 5`

#### In `build.zig`

First, to fetch the dependency, use

```
zig fetch --save git+https://github.com/jpwol/zopt
```

Note that `--save` will automatically update your `build.zig.zon`, but may be omitted.

In `build.zig`,

```zig

const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const zopt = b.dependency("zopt", .{
        .target = target,
        .optimize = optimize,
    });

    const zopt_mod = zopt.module("zopt");

    const main_mod = b.createModule(.{
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    main_mod.addImport("zopt", zopt_mod);

    const exe = b.addExecutable(.{
        .name = "test",
        .root_module = mod,
    });

    b.installArtifact(exe);
```

where `main_mod` is just an example of your main module. The main takeaway is once you fetch the dependency,
it can be defined in your `build.zig` with `b.dependency(...)`, and the module for importing is extracted with `.module(...)`.
