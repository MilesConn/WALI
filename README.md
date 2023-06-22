# Webassembly Linux Interface (WALI)

This repo serves to document design decisions, requirements, and specifications for the WALI.

## Overview
We create a custom modified [musl libc](https://github.com/arjunr2/wali-musl) and produce a baseline
implementation in [WAMR](https://github.com/SilverLineFramework/wasm-micro-runtime/tree/wali)

## Getting Started

There are three major components for testing: WALI libc, WALI runtime, and the test suite. All three of these can
be built with the default `make` command. However, initial setup and packages may be required for some of the components

### Prerequisites

Tested with LLVM Clang 16. Requires *compiler-rt* builtins from LLVM 16 for wasm32 for full libc support


### Building WALI libc

The [wali-musl](wali-musl) submodule has detailed information on prerequisites and steps for compiling libc

Once the initial setup is performed for building libc, you may use the makefile target:
```shell
make libc
```

NOTE: We currently support only x86-64 but future work will include support for other architectures.

### Building WALI runtime

We produce a baseline implementation in [WAMR](https://github.com/SilverLineFramework/wasm-micro-runtime/tree/wali).
For details on how to implement these native APIs in WAMR, refer [here](https://github.com/bytecodealliance/wasm-micro-runtime/blob/main/doc/export_native_api.md)

After cloning the wasm-micro-runtime submodule, the following target will build the WALI implementation of the runtime
```shell
make iwasm
```
You may add the *IWASM_DIR* in the Makefile to your PATH to access the *iwasm* executable.


### Compiling Test Suite
```shell
make tests
```

The above target builds all the C files in [tests](tests) and generates the WASM executables `tests/wasm/*_link.wasm`, which
can be executed by a WALI runtime. It also generates native ELF files for the respective tests in `tests/elf` to compare
against the WASM output


### Running WASM code

Use a Webassembly runtime of your choice to execute the above generated WASM code.

If you built the baseline WAMR implementation and added the *IWASM_DIR* from the Makefile to your path,
you can use `iwasm <path-to-wasm-file>` to execute the code.


## Compilation Steps

Note that Clang 16 shipped with WALI is required. After adding to path, to compile C to WASM, refer to
[compile-wali-standalone.sh](tests/compile-wali-standalone.sh), and run:

```shell
# Compile standalone C file
clang \
  --target=wasm32-wasi-threads -O3  \
  `# Sysroot and lib search path` \
  --sysroot=<path-to-wali-sysroot> -L<path-to-wali-sysroot>/lib \
  `# Enable wasm extension features`  \
  -matomics -mbulk-memory -mmutable-globals -msign-ext  \
  `# Linker flags for shared mem + threading` \
  -Wl,--shared-memory -Wl,--export-memory -Wl,--max-memory=67108864 \
  <input-c-file> -o <output-wasm-file>
```


Since changes are yet to be made to `clang/wasm-ld` for the wali toolchain, we are using support enabled 
in `wasi-threads` target. This will change once a `wasm32-linux` target is added for WALI.

For indepedent compilation and linking, refer to [compile-wali.sh](tests/compile-wali.sh) in the test suite compilation toolchai


## Resources
[Syscall Information Table](https://docs.google.com/spreadsheets/d/1__2NqMqGLHdjFFYonkF49IkGgfv62TJCpZuXqhXwnlc/edit?usp=sharing)

This paper (https://cseweb.ucsd.edu/~dstefan/pubs/johnson:2022:wave.pdf) and its related work section, especially the bit labeled "Modeling and verifying system interfaces"

