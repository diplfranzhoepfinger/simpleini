# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

SimpleIni is a **header-only C++ library** (`SimpleIni.h`) for reading and writing INI-style configuration files. The entire library is a single header; no build step is needed to use it.

## Build and test

```bash
# Configure (downloads GoogleTest via FetchContent by default)
cmake -S . -B build

# Build the test binary
cmake --build build

# Run all tests
ctest --verbose --test-dir build

# Skip tests
cmake -S . -B build -DBUILD_TESTING=OFF

# Use system GoogleTest instead of downloading
cmake -S . -B build -DSIMPLEINI_USE_SYSTEM_GTEST=ON
```

In-source builds are blocked by CMake. Always use a separate build directory.

To run a single GoogleTest test case:

```bash
./build/tests/tests --gtest_filter="TestSuiteName.TestName"
```

## Architecture

The entire library lives in `SimpleIni.h`. The main class is the template `CSimpleIniTempl<SI_CHAR, SI_STRLESS, SI_CONVERTER>`. Common typedefs:

| Typedef | Char type | Case-sensitive |
|---|---|---|
| `CSimpleIniA` | `char` | No |
| `CSimpleIniCaseA` | `char` | Yes |
| `CSimpleIniW` | `wchar_t` | No |
| `CSimpleIniCaseW` | `wchar_t` | Yes |

**Converter selection** (defined before `#include "SimpleIni.h"`):

- `SI_NO_CONVERSION` — default on Linux/Mac; no encoding conversion, UTF-8 pass-through
- `SI_CONVERT_GENERIC` — cross-platform UTF-8/MBCS using `<uchar.h>` (`mbrtoc32`/`c32rtomb`); requires a UTF-8 locale for non-ASCII (`setlocale(LC_ALL, "")`)
- `SI_CONVERT_WIN32` — default on Windows; uses Win32 API
- `SI_CONVERT_ICU` — uses ICU library; requires ICU headers and `icuuc.lib`

**Optional feature macros:**

- `SI_SUPPORT_IOSTREAMS` — enables STL stream support (open file with `ios_base::binary`)
- `SI_NO_MBCS` — disables `<mbstring.h>` on Windows

**Multi-line values** use heredoc syntax in the INI file: `key = <<<ENDTAG ... ENDTAG`.

**Return values:** `SI_Error` is negative on error, `SI_OK` (0) on success, `SI_INSERTED` / `SI_UPDATED` (positive) from `SetValue`.

## Tests

Each `tests/ts-*.cpp` file covers a specific area:

| File | Coverage |
|---|---|
| `ts-snippets.cpp` | API usage examples from README |
| `ts-bugfix.cpp` | Regression tests for past bugs |
| `ts-roundtrip.cpp` | Load → save → reload fidelity |
| `ts-utf8.cpp` | UTF-8 encoding |
| `ts-generic.cpp` | `SI_CONVERT_GENERIC` (mbrtoc32/c32rtomb) |
| `ts-wchar.cpp` | `wchar_t` interface (Windows only) |
| `ts-numeric.cpp`, `ts-boolean.cpp` | Typed value helpers |
| `ts-multiline.cpp`, `ts-quotes.cpp` | Format edge cases |
| `ts-casesensitivity.cpp`, `ts-sections.cpp`, `ts-deletion.cpp`, `ts-edgecases.cpp`, `ts-noconvert.cpp` | API behaviour |

Tests load `tests.ini` (and `example.ini`) from the binary directory; the CMake post-build step copies them there automatically.

## Making a release

1. Update version in `SimpleIni.h`
2. Update version in `CMakeLists.txt`
3. Commit, then create a GitHub release with a tag like `v4.25`
