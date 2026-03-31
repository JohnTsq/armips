# AGENTS.md

## Build & Install

```bash
# Configure a separate build directory (recommended)
cmake -S . -B build                # Detect compiler, apply options

# Build (default Release on Unix, Debug/Release on Windows)
cmake --build build --config Release   # Windows: add "--config Debug" for a debug build

# Optional: install the binaries to a custom prefix
cmake --install build --prefix <install_dir>
```

*The repository builds three targets:*  
- `armips-bin` – the assembler executable (`armips`).  
- `armipstests` – the test runner (built only when `ARMIPS_LIBRARY_ONLY` is OFF).  
- `armips` – static library (`libarmips`).

You can control optional features via CMake cache variables:

```bash
-DAARMIPS_PRECOMPILE_HEADERS=ON   # Enable pre‑compiled headers
-DAARMIPS_REGEXP=ON               # Enable regexp expression functions (default: ON)
-DAARMIPS_USE_STD_FILESYSTEM=ON   # Use std::filesystem instead of ghc::filesystem
-DAARMIPS_LIBRARY_ONLY=ON         # Build only the static library, skip binaries
```

## Running the Assembler

```bash
# General usage (see README for full option list)
./build/armips-bin <source.asm> [options]
```

Typical options include `-temp <file>`, `-sym <file>`, `-sym2 <file>`, `-erroronwarning`, etc.

## Test Suite

The test runner is built as `armipstests`. It expects a single argument – the path to the **Tests** folder – and will recursively execute every `*.asm` test found beneath that directory.

### Run all tests (CMake/CTest)
```bash
# Build already performed – now execute the CTest entry
ctest --test-dir build --output-on-failure   # runs the single CTest called "armipstests"
```

### Run all tests directly (alternative)
```bash
./build/armipstests Tests   # "Tests" is the repository root test directory
```

### Run a subset / single test
Pass the sub‑directory that contains the desired test(s). The runner will only execute the tests in that subtree.
```bash
# Example: run only the size‑related tests
./build/armipstests Tests/Region/Size
```

If you need to isolate a single `.asm` file, copy it (or a small folder containing it) to a temporary directory and point the runner at that directory.

## Lint / Formatting

The project does not ship a dedicated lint target, but a **clang‑format** configuration is provided for the `ext/filesystem` submodule. The root source follows a similar style (4‑space indentation, 256‑column limit, Chromium‑based Allman braces).

```bash
# Format the whole codebase (requires clang‑format ≥ 9)
clang-format -i $(git ls-files "*.cpp" "*.h")

# Use the submodule‑specific config for those files:
clang-format -style=file -i ext/filesystem/**/*.cpp ext/filesystem/**/*.h
```

Feel free to add a `format` target to `CMakeLists.txt` if you need a one‑liner:
```cmake
add_custom_target(format
    COMMAND clang-format -i ${ALL_SOURCE_FILES}
    COMMENT "Running clang‑format on the source tree"
)
```

## Code‑Style Guidelines

These conventions are inferred from the existing codebase and should be followed for new contributions.

1. **General Layout**
   - Header guards: use `#pragma once`.
   - Include order: project headers first (quoted, path relative to repository root), then standard library headers.
   - No trailing whitespace; line endings are LF on Unix, CRLF on Windows.
   - Maximum line length: 256 characters (as enforced by `.clang-format`).

2. **Indentation & Braces**
   - 4 spaces per indentation level, no tabs.
   - Brace style follows the *Chromium* Allman variant (see `.clang-format`):
     ```cpp
     class Foo {
     public:
         Foo();
     };
     ```
   - `else`, `catch`, `while` etc. start on a new line with the opening brace on the same line as the clause.

3. **Naming Conventions**
   - **Macros / Constants**: `UPPER_SNAKE_CASE` (e.g., `ARM_SHIFT_LSL`).
   - **Enums**: plain `enum` with PascalCase names, values in `UPPER_SNAKE`.
   - **Types / Classes / Structs**: `PascalCase` (e.g., `CArmArchitecture`).
   - **Functions & Methods**: `camelCase` for regular functions, `PascalCase` for getters/setters that expose a boolean/status (e.g., `SetThumbMode`).
   - **Variables**: `lowerCamelCase` (e.g., `testName`, `errorString`).
   - **Namespaces**: The code does not currently use explicit namespaces; if added, use lower‑case names.

4. **Header Files**
   - Use `#pragma once` instead of traditional include guards.
   - Forward declare types when possible to reduce compile dependencies.
   - Keep public API minimal; implementation details stay in source files.

5. **Memory Management**
   - Prefer RAII: `std::unique_ptr`, `std::shared_ptr`, or stack‑allocated objects.
   - Avoid raw `new`/`delete`. When interfacing with legacy APIs, manage ownership explicitly.

6. **Error Handling**
   - Functions that can fail return `bool` and accept an output `std::wstring& errorString` (or similar) to convey diagnostics.
   - Use early returns for error cases to reduce nesting.
   - When a test expects a specific exit code, encode it in `commandLine.txt` as the first token (as used by `TestRunner`).

7. **Standard Library Usage**
   - Prefer `std::vector`, `std::string`/`std::wstring`, `std::unique_ptr`.
   - Use range‑for loops where appropriate.
   - Use `constexpr` for compile‑time constants when possible.

8. **C++ Version**
   - The project targets **C++17** (enforced by `CXX_STANDARD 17`).
   - Use language features supported by GCC 9+, Clang 7+, MSVC 2017+ (e.g., structured bindings, `std::optional` if needed).

9. **Threading**
   - The build adds the `Threads` library on non‑Windows platforms; otherwise rely on OS‑specific primitives.

10. **Testing Conventions**
    - Each test lives under `Tests/<category>/<sub‑category>/` with a matching `<name>.asm` and optional `expected.txt` / `expected.bin` / `commandLine.txt` files.
    - Test names are case‑insensitive; they must not contain spaces.
    - New tests should be added to the appropriate folder and follow the same file‑pair layout.

## Cursor / Copilot Rules

No `.cursor` or `.cursorrules` directory nor `.github/copilot‑instructions.md` were found in the repository, so there are no additional agent‑specific constraints.

---
*Generated for agentic tooling to provide consistent build, test, lint, and style guidance.*
