## 2026-10-09

### `ptrtoint` Problems

After removing function pointers, the next thing that Gordium got stuck on is casting a pointer to an int. That was easy for AI to solve since it only was used to keep track of index/remaining items in an array.

### Allocation Problems

Next, there was a panic on an "unreachable part of the code".

```rust
let struct_id = match module.lab_stype(struct_typelab) {
    STypeFrag::Struct(s) => StructIdent(s.clone()),
    _ => unreachable!("`struct_typelab` should have a struct type"),
};
```

According to AI, this is occurring because of uses of `calloc` and other variants of allocating functions, which Gordium does not yet handle (Gordium only overrides `malloc`).

AI proposes making a monomorphized wrapper for these kinds of functions. This appears to have worked. Now constraint generation runs out of memory.

## 2026-10-07

I just got Codex, now I'm going to be trying to run Gordium on [giflib](https://sourceforge.net/p/giflib/code/ci/master/tarball?path).

First, I asked Codex to figure out how to create a `source.c` file for `giflib`. It proposed putting all of the library sources together in one place:

```c
#include "dgif_lib.c"
#include "egif_lib.c"
#include "gifalloc.c"
#include "gif_err.c"
#include "gif_font.c"
#include "gif_hash.c"
#include "openbsd-reallocarray.c"
#include "quantize.c"
```

I tried this, then ran

```sh
cargo run -- giflib
```

but I got an error

```plaintext
Error: "The file '\"giflib/source.c\"' is missing in '\"giflib\"'."
```

I'm not sure why this is happening, since I put the above code in `source.c`.

---

According to AI, it's because the new code accepts a path to the directory, rather than just the name of the benchmark. Running

```sh
cargo run -- benchmarks/giflib
```

worked.

### Compiler Problems Again... 

It now produces the following error:

```plaintext
Error: "clang-17 failed with status exit status: 1:\nIn file included from benchmarks/giflib/source.c:1:\nIn file included from benchmarks/giflib/dgif_lib.c:13:\nIn file included from /usr/include/fcntl.h:25:\nIn file included from /usr/include/features.h:548:\n/usr/include/gnu/stubs.h:7:11: fatal error: 'gnu/stubs-32.h' file not found\n    7 | # include <gnu/stubs-32.h>\n      |           ^~~~~~~~~~~~~~~~\n1 error generated.\n"
```

When "de-stringified", the error looks like this:

```plaintext
clang-17 failed with status exit status: 1:
In file included from benchmarks/giflib/source.c:1:
In file included from benchmarks/giflib/dgif_lib.c:13:
In file included from /usr/include/fcntl.h:25:
In file included from /usr/include/features.h:548:
/usr/include/gnu/stubs.h:7:11: fatal error: 'gnu/stubs-32.h' file not found
    7 | # include <gnu/stubs-32.h>
      |           ^~~~~~~~~~~~~~~~
1 error generated.
```

This has to do with the fact that I'm running on `x86_64` instead of `aarch64`. See [[2026-09-18#Compilation problems|my previous notes on this problem]] for more details.

---

Working with AI on the compilation problem, it gave me this script to run in order to install the ARM64 toolchain (derived from [here](https://learn.arm.com/install-guides/gcc/arm-gnu/)):

```sh
mkdir -p /tmp/arm-sdk

curl -fL \
  https://developer.arm.com/-/media/Files/downloads/gnu/14.2.rel1/binrel/arm-gnu-toolchain-14.2.rel1-x86_64-aarch64-none-linux-gnu.tar.xz \
  -o /tmp/arm-sdk/toolchain.tar.xz

tar -xJf /tmp/arm-sdk/toolchain.tar.xz -C /tmp/arm-sdk
```

I could then use the `compile.sh` script to compile to `source.ll`.

```sh
ARM_TOOLCHAIN=/tmp/arm-sdk/arm-gnu-toolchain-14.2.rel1-x86_64-aarch64-none-linux-gnu
ARM_SYSROOT="$("$ARM_TOOLCHAIN/bin/aarch64-none-linux-gnu-gcc" -print-sysroot)"

./compile.sh giflib --sysroot="$ARM_SYSROOT"
cargo run -- benchmarks/giflib
```

If I wanted to install the toolchain as a permanent toolchain, I can run

```sh
ARM_TOOLCHAIN=/tmp/arm-sdk/arm-gnu-toolchain-14.2.rel1-x86_64-aarch64-none-linux-gnu
ARM_SYSROOT="$("$ARM_TOOLCHAIN/bin/aarch64-none-linux-gnu-gcc" -print-sysroot)"

sudo mkdir -p /usr/aarch64-linux-gnu/sys-root
sudo cp -a "$ARM_SYSROOT"/. /usr/aarch64-linux-gnu/sys-root/
```

I would still have to pass it as the sysroot, though:

```sh
./compile.sh giflib --sysroot=/usr/aarch64-linux-gnu/sys-root
cargo run -- benchmarks/giflib
```

---

This appears to work.

### Removing Function Pointers

The first problem that Gordium ran into is function pointers.

```plaintext
thread 'main' (29217) panicked at src/lang/instruction.rs:363:18:
 ERR: Function pointers unsupported.
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

AI found two uses of function pointers. The first is for **I/O**. It has two types that represent function pointers:

```c
/* func type to read gif data from arbitrary sources (TVT) */
typedef int (*InputFunc)(GifFileType *, GifByteType *, int);

/* func type to write gif data to arbitrary targets.
 * Returns count of bytes written. (MRB)
 */
typedef int (*OutputFunc)(GifFileType *, const GifByteType *, int);
```

These are used in two functions:

```c
GifFileType *DGifOpen(void *userPtr, InputFunc readFunc,
                      int *Error); /* new one (TVT) */

/* ... and later ... */

GifFileType *EGifOpen(void *userPtr, OutputFunc writeFunc, int *Error);
```

The second use of function pointers is when a sorting function is passed to `qsort`.

```c
qsort(SortArray, NewColorSubdiv[Index].NumEntries,
      sizeof(QuantizedColorType *), SortCmpRtn);
//                                  ^^^^^^^^^^
```
 
 The first use (I/O), we may be able to get rid of by replacing it with an enum with possible options. For the second use, we may need to inline the `qsort` function from `stlib` with the compare function hard coded in.

> [!success]
> AI ended up doing this. It implemented `qsort` from `musl`, and it removed the function pointer parameters since the binaries always pass `NULL` in.