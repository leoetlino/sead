# sead

This is a decompilation of sead, the standard C++ library for first-party Nintendo games.

Unlike the original [sead decompilation project](https://github.com/aboood40091/sead), which this repo derives from, this project targets more recent versions of sead.

The objective is to recreate the standard library as accurately as possible, so that interoperability can eventually be achieved by adding support for other platforms and by making it easier to create projects that interact with sead games.

Because sead is statically linked into games, it is required to legally own at least one recent first-party Nintendo game. Picking a game that ships with debugging symbols is strongly recommended:

* Super Mario Odyssey (version 1.0.0) ([buy it here](https://www.nintendo.com/games/detail/super-mario-odyssey-switch/))
* Splatoon 2 (version <= 3.1.0) ([buy it here](https://www.nintendo.com/games/detail/splatoon-2-switch/))
* [Nintendo Labo](https://labo.nintendo.com/) (the pilot build has symbols, file names and assertions)
* Any other title that has symbols and uses sead

File names, function names and the file organization come from debugging symbols, assertions and information in all of the aforementioned titles.
Nobody except Nintendo has the source code of sead, not even third-party developers.

Note that many names (especially for inlined, templated functions) are just plain guesses.

## Modules

For progress, refer to [the GitHub project page](https://github.com/zeldamods/sead/projects/1). Several modules currently fail to build for Switch.

* **audio** - Audio
* **basis** - Types, asserts, allocation operators
* **codec** - Base64, CRC16, CRC32
* **container** - Templated container classes
* **controller** - Controller
* **devenv** - Development environment (debug utilities)
* **filedevice** - File IO
* **framework** - Framework (game framework, tasks, etc.)
* **geom** - Geometry
* **gfx** - Graphics
* **heap** - Heap (arenas, disposers, different types of heaps)
* **hostio** - Host IO (communication with PCs)
* **math** - Maths utilities (vector, matrix, etc.)
* **message** - libms wrapper
* **mc** - Multi-core support
* **prim** - Primitives (strings, enums, RTTI, etc.)
* **random** - Random number generator
* **resource** - Resource (loading, decompressing, etc.)
* **stream** - Stream IO
* **thread** - Thread utilities (threads, critical sections, message queues, etc.)
* **time** - Time utilities

### Platform specific source

Platform-specific files are usually placed into a subdirectory that is called:

* **cafe** for Wii U
* **ctr** for 3DS
* **nin** for nnSdk (Switch)

## Building

Building this project requires:

- A C++17 capable compiler (or >= Clang 4.0). While older parts of sead are written in C++03, the newer modules in sead target C++11 (or newer) and recent C++ language or library features make writing C++ more convenient.
- CMake 3.10+

### Configuration

sead can be configured with several compile-time defines:

* `SEAD_DEBUG`: enables assertions and HostIO code.

#### Platforms
* `cafe`: Wii U
* `NNSDK`: nnSdk (Switch and any platform that is supported by modern nnSdk)

Other platforms (generic Unix, iOS, Android, CTR) are not supported.

#### Matching hacks

This project sometimes uses small hacks to force particular code to be generated by the compiler. Those have no semantic effects but can help with matching assembly code especially when the hacks are used for functions that are inlined.

* `MATCHING_HACK_NX_CLANG`: Hacks for Switch sead, when compiling with Clang.

## Contributing

### Non-inlined functions
When **implementing non-inlined functions**, please compare the assembly output against the original function and make it match the original code. At this scale, that is pretty much the only reliable way to ensure accuracy and functional equivalency.

However, given the large number of functions, certain kinds of small differences can be ignored when a function would otherwise be equivalent:

* Regalloc differences.

* Instruction reorderings when it is obvious the function is still semantically equivalent (e.g. two add/mov instructions that operate on entirely different registers being reordered)

When ignoring minor differences, add a `// NOT_MATCHING: explanation` comment and explain what does not match.

### Header utilities or inlined functions
For **header-only utilities** (like container classes), use pilot/debug builds, assertion messages and common sense to try to undo function inlining. For example, if you see the same assertion appear in many functions and the file name is a header file, or if you see identical snippets of code in many different places, chances are that you are dealing with an inlined function. In that case, you should refactor the inlined code into its own function.

Also note that introducing inlined functions is sometimes necessary to get the desired codegen.

If a function is inlined, you should try as hard as possible to make it match perfectly. For inlined functions, it is better to use weird code or small hacks to force a match as differences would otherwise appear in every single function that inlines the non-matching code, which drastically complicates matching other functions. If a hack is used, wrap it inside a `#ifdef MATCHING_HACK_{PLATFORM}` (see above for a list of defines).
