# libsxcli2

Small C++17 static CLI library extracted from [Smithproxy](https://github.com/astibal/smithproxy)
master at `ae0415f69624618d7c611ff7a336280352a50f5b`.

- Hierarchical commands, unique prefixes, quoted arguments, validation and help.
- Contextual completion, command availability and confirmation callbacks.
- Terminal line editing, history, cursor movement and ANSI color themes.
- Movable, owning or borrowing file-descriptor transport for sockets and stdio.

The library produces `libsxcli2.a` and has no Smithproxy, Socle or third-party
runtime dependencies. It uses C++17 and POSIX APIs (Linux/Unix).
Existing `libcli2::` names and both `cli.hpp` and `service/libcli/cli.hpp`
include styles are preserved. Application commands and configuration schemas
belong in the consuming project.

## Build

```sh
# Debian / Ubuntu; GoogleTest is needed only for tests.
sudo apt-get install cmake g++ libgtest-dev
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
ctest --test-dir build --output-on-failure
./build/sxcli2-demo
cmake --install build --prefix "$PWD/install"
```

CMake >= 3.16 is required. `SXCLI2_BUILD_TESTS` and `SXCLI2_BUILD_EXAMPLES`
default to ON for standalone builds and OFF when embedded.

## Integrate

```sh
git submodule add https://github.com/astibal/libsxcli2.git external/libsxcli2
```

```cmake
add_subdirectory(external/libsxcli2)
target_link_libraries(your_target PRIVATE sxcli2::sxcli2)
```

For an installed package, replace `add_subdirectory` with
`find_package(sxcli2 CONFIG REQUIRED)` and set `CMAKE_PREFIX_PATH` to its prefix.
The compatibility target `smithproxy_libcli` is also available.

```cpp
#include <service/libcli/cli.hpp>

libcli2::Cli cli;
cli.command("show session")
    .help("Show one session")
    .argument({"id", "Session identifier"})
    .handler([](libcli2::Context& context, const libcli2::Invocation& call) {
        context.print("session: " + call.arguments.at(0));
        return 0;
    });
```

When migrating Smithproxy, replace its `smithproxy_libcli` target definition,
remove the duplicate library sources/headers and retain application-specific
`src/service/cmd` code. Pin the submodule to a reviewed commit; port any
unmerged consumer changes before removing local copies.

## Provenance and license

The library, interactive example and two existing test files were copied
unchanged from the Smithproxy commit above. Standalone CMake packaging and this
README were added. The original GPLv3 [LICENSE](LICENSE) is preserved.
