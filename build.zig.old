const std = @import("std");

/// Directories with our includes.
const root = thisDir() ++ "../../../vendor/freetype/";
const include_path = root ++ "include";
pub const include_path_self = thisDir();

pub const include_paths = .{ include_path, include_path_self };

pub fn module(b: *std.Build) *std.build.Module {
    return b.createModule(.{
        .source_file = .{ .path = (comptime thisDir()) ++ "/main.zig" },
    });
}

fn thisDir() []const u8 {
    return std.fs.path.dirname(@src().file) orelse ".";
}

pub const Options = struct {
    libpng: Libpng = .{},
    zlib: Zlib = .{},

    pub const Libpng = struct {
        enabled: bool = false,
        step: ?*std.build.LibExeObjStep = null,
        include: ?[]const []const u8 = null,
    };

    pub const Zlib = struct {
        enabled: bool = false,
        step: ?*std.build.LibExeObjStep = null,
        include: ?[]const []const u8 = null,
    };
};

pub fn link(
    b: *std.Build,
    step: *std.build.LibExeObjStep,
    opt: Options,
) !*std.build.LibExeObjStep {
    const lib = try buildFreetype(b, step, opt);
    step.linkLibrary(lib);
    step.addIncludePath(.{ .path = include_path });
    step.addIncludePath(.{ .path = include_path_self });
    return lib;
}

pub fn buildFreetype(
    b: *std.Build,
    step: *std.build.LibExeObjStep,
    opt: Options,
) !*std.build.LibExeObjStep {
    const target = step.target;
    const lib = b.addStaticLibrary(.{
        .name = "freetype",
        .target = target,
        .optimize = step.optimize,
    });

    // Include
    lib.addIncludePath(.{ .path = include_path });

    // Link
    lib.linkLibC();
    if (opt.libpng.enabled) {
        if (opt.libpng.step) |libpng|
            lib.linkLibrary(libpng)
        else
            lib.linkSystemLibrary("libpng");

        if (opt.libpng.include) |dirs|
            for (dirs) |dir| lib.addIncludePath(.{ .path = dir });
    }
    if (opt.zlib.enabled) {
        if (opt.zlib.step) |zlib|
            lib.linkLibrary(zlib)
        else
            lib.linkSystemLibrary("z");

        if (opt.zlib.include) |dirs|
            for (dirs) |dir| lib.addIncludePath(.{ .path = dir });
    }

    // Compile
    var flags = std.ArrayList([]const u8).init(b.allocator);
    defer flags.deinit();

    try flags.appendSlice(&.{
        "-DFT2_BUILD_LIBRARY",

        "-DHAVE_UNISTD_H",
        "-DHAVE_FCNTL_H",

        "-fno-sanitize=undefined",
    });
    if (opt.libpng.enabled) try flags.append("-DFT_CONFIG_OPTION_USE_PNG=1");
    if (opt.zlib.enabled) try flags.append("-DFT_CONFIG_OPTION_SYSTEM_ZLIB=1");

    // C files
    lib.addCSourceFiles(srcs, flags.items);
    switch (target.getOsTag()) {
        .linux => lib.addCSourceFile(.{
            .file = .{ .path = root ++ "builds/unix/ftsystem.c" },
            .flags = flags.items,
        }),
        .windows => lib.addCSourceFile(.{
            .file = .{ .path = root ++ "builds/windows/ftsystem.c" },
            .flags = flags.items,
        }),
        else => lib.addCSourceFile(.{
            .file = .{ .path = root ++ "src/base/ftsystem.c" },
            .flags = flags.items,
        }),
    }
    switch (target.getOsTag()) {
        .windows => {
            lib.addCSourceFiles(&.{
                root ++ "builds/windows/ftdebug.c",
                root ++ "src/base/ftver.c",
            }, flags.items);
        },
        else => lib.addCSourceFile(.{
            .file = .{ .path = root ++ "src/base/ftdebug.c" },
            .flags = flags.items,
        }),
    }

    return lib;
}

const srcs = &.{
    root ++ "src/autofit/autofit.c",
    root ++ "src/base/ftbase.c",
    root ++ "src/base/ftbbox.c",
    root ++ "src/base/ftbdf.c",
    root ++ "src/base/ftbitmap.c",
    root ++ "src/base/ftcid.c",
    root ++ "src/base/ftfstype.c",
    root ++ "src/base/ftgasp.c",
    root ++ "src/base/ftglyph.c",
    root ++ "src/base/ftgxval.c",
    root ++ "src/base/ftinit.c",
    root ++ "src/base/ftmm.c",
    root ++ "src/base/ftotval.c",
    root ++ "src/base/ftpatent.c",
    root ++ "src/base/ftpfr.c",
    root ++ "src/base/ftstroke.c",
    root ++ "src/base/ftsynth.c",
    root ++ "src/base/fttype1.c",
    root ++ "src/base/ftwinfnt.c",
    root ++ "src/bdf/bdf.c",
    root ++ "src/bzip2/ftbzip2.c",
    root ++ "src/cache/ftcache.c",
    root ++ "src/cff/cff.c",
    root ++ "src/cid/type1cid.c",
    root ++ "src/gzip/ftgzip.c",
    root ++ "src/lzw/ftlzw.c",
    root ++ "src/pcf/pcf.c",
    root ++ "src/pfr/pfr.c",
    root ++ "src/psaux/psaux.c",
    root ++ "src/pshinter/pshinter.c",
    root ++ "src/psnames/psnames.c",
    root ++ "src/raster/raster.c",
    root ++ "src/sdf/sdf.c",
    root ++ "src/sfnt/sfnt.c",
    root ++ "src/smooth/smooth.c",
    root ++ "src/svg/svg.c",
    root ++ "src/truetype/truetype.c",
    root ++ "src/type1/type1.c",
    root ++ "src/type42/type42.c",
    root ++ "src/winfonts/winfnt.c",
};
