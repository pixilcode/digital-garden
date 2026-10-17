## 2026-11-18

> [!question]
> Are there uses of `unsafe` in `giflib-rs`? If so, where are they? And how do they impact the interface?

It looks like there aren't any uses of `unsafe` that significantly affect whether a variable is mutable or not.

> [!question]
> Why does Gordium infer a slice for the field in `GifColorType` when the type is not a slice?

The field name is `Image`.

In most cases, the field is accessed only to access subfields. For example,

```c
if (GifFile->Image.ColorMap) {
	GifFreeMapObject(GifFile->Image.ColorMap);
	GifFile->Image.ColorMap = NULL;
}
```

However, there is one case where it is used with `memcpy`.

```c
memcpy(&sp->ImageDesc, &GifFile->Image, sizeof(GifImageDesc));
```

I'm not sure exactly why this would infer a box slice. The corresponding LLVM code is

```llvm
%ImageDesc = getelementptr inbounds %struct.SavedImage, ptr %arrayidx, i32 0, i32 0
%Image = getelementptr inbounds %struct.GifFileType, ptr %GifFile, i32 0, i32 7
call void @llvm.memcpy.p0.p0.i64(ptr align 8 %ImageDesc, ptr align 8 %Image, i64 32, i1 false)
```

which translates to the stypes

```stypes
[740]    [ptr([struct(struct.GifImageDesc)]⁸⁰⁵⁴)]⁸⁰⁵³ ImageDesc = [&*(*arrayidx).0]⁸⁹⁹ => ptr;
[741]    [ptr([struct(struct.GifImageDesc)]⁸⁰⁵⁶)]⁸⁰⁵⁵ Image = [&*(*GifFile).7]⁹⁰⁰ => ptr;
[742]    llvm.memcpy.p0.p0.i64([ImageDesc]⁹⁰¹ => ptr, [Image]⁹⁰² => ptr, [(const expr)]⁹⁰³ => i64, [(const expr)]⁹⁰⁴ => bool, );
```

### Side adventure: what is the "grammar" for our source language?

```ebnf
si_instr ::=
    | 'alloca' var_ident
    | 'load' var_ident var_ident type_lab
    | 'store' var_ident [ var_ident ] type_lab
    | 'field' var_ident var_ident type_lab struct_id field_idx
    | 'element' 

si_type ::= { s_type_frag }

s_type_frag ::=
    | 'void'
    | 'bool'
    | 'i8'
    | 'i16'
    | 'i32'
    | 'i64'
    | 'f32'
    | 'f64'
    | 'struct(' STRING ')'
    | 'ptr'
    | 'unknown'
    
struct ::= 'struct(' struct_ident ') {' { '<' { type_lab ',' } '>,' } '}'

type_lab ::= U64
var_ident ::= USIZE
struct_ident ::= STRING
```

## 2026-11-04

Here's a comparison of the results of Gordium and `giflib-rs`.

### Doesn't understand the string type

I'm sure this is a known problem, but it doesn't understand C strings.

```diff
fn DGifOpenFileName(
-   FileName: <shared<[]>, i8, >,
+   FileName: <shared, unknown, >,
    Error: <mut, i32, >,
) -> <Rc, struct(struct.GifFileType), >;

fn EGifGCBToExtension(
    GCB: <shared, struct(struct.GraphicsControlBlock), >,
-   GifExtension: <mut<[]>, i8, >,
+   GifExtension: <mut<[]>, unknown, >,
) -> <i64, >;
```

### Not understanding `GifColorType`

Gordium appears not to handle `GifByteType` correctly?

As an example, `GifColorType` has three fields, that are all essentially `u8`s.

```c
typedef unsigned char GifByteType;

/* ... elsewhere ... */

typedef struct GifColorType {
	GifByteType Red, Green, Blue;
} GifColorType;
```

Gordium infers `GifColorType`'s type as

```plaintext
struct struct.GifColorType (
    <ghost, unknown, >,
    <ghost, unknown, >,
    <ghost, unknown, >,
);
```

when it could infer

```plaintext
struct struct.GifColorType (
    <ghost, i8, >,
    <ghost, i8, >,
    <ghost, i8, >,
);
```

This seems to be a pretty consistent pattern anytime `GifByteType` is used.

### Treats a field as slice when type is not a slice

In `GifFileType`, there's a field `Image` of type `GifImageDesc`. Gordium infers the type of this field to be `<Box<[]>, struct(struct.GifImageDesc)>`, even though it's not even a pointer.

```diff
struct struct.GifFileType (
    /* ... */
-   <ghost, struct(struct.GifImageDesc), >,
+   <Box<[]>, struct(struct.GifImageDesc), >,
    /* ... */
);
```

I'm not exactly sure why this is. Most everywhere that it's used, the code directly accesses one of the struct's fields. The only place where this isn't the case is when it's copied.

```c
memcpy(&sp->ImageDesc, &GifFile->Image, sizeof(GifImageDesc));
```

### Infers `GifFileType` as internally mutable

> [!summary]
> It appears that Gordium infers the `GifFileType` to be a struct that can be passed around in an `Rc`, then passes it around as immutable (`Rc` or `shared`). There are some times that it needs mutability, so it somehow infers `Rc<Cell>`, though I don't think there's a way to get `Rc<Cell>` from `Rc`.
> 
> `giflib-rs` instead stores `GifFileType` in a `Box` and passes around `mut` references to it.
> 
> It appears more generally that Gordium prefers internally mutable over passing around mutable references

#### Type tradeoffs

Gordium appears to be inferring less precise types than `giflib-rs` has for the `GifFileType` struct. For example, here's the (relevant) differences between `GifFileType` for `giflib` (`+`) and `giflib-rs` (`-`):

```diff
struct struct.GifFileType (
    <ghost, i32, >,
    <ghost, i32, >,
    <ghost, i32, >,
    <ghost, i32, >,
    <ghost, unknown, >,
-   <ghost, Box, struct(struct.ColorMapObject), >,
+   <ghost, Rc<Cell>, struct(struct.ColorMapObject), >,
    <ghost, i32, >,
-   <ghost, struct(struct.GifImageDesc), >,
+   <Box<[]>, struct(struct.GifImageDesc), >,
-   <ghost, mut<[]>, struct(struct.SavedImage), >,
+   <ghost, Rc<[Cell]>, struct(struct.SavedImage), >,
    <ghost, i32, >,
-   <ghost, mut<[]>, struct(struct.ExtensionBlock), >,
+   <ghost, Rc<[]>, struct(struct.ExtensionBlock), >,
-   <ghost, i32, >,
+   <ghost<Cell>, i32, >,
    <ghost, shared, unknown, >,
-   <ghost, Box, struct(struct.GifFilePrivateType), >,
+   <ghost, shared<Cell>, struct(struct.GifFilePrivateType), >,
);
```

However, it appears like the differences in this struct type allow for a more precise type in the function definitions. For example, `DGifBufferedInput`, `DGifDecompressInput`, and `DGifDecompressLine`:

```diff
fn DGifBufferedInput(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
    Buf: <mut<[]>, unknown, >,
    NextByte: <mut, unknown, >,
) -> <i32, >;

fn DGifDecompressInput(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
    Code: <mut, i32, >,
) -> <i32, >;

fn DGifDecompressLine(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
    Line: <mut<[]>, i8, >,
    LineLen: <i32, >,
) -> <i32, >;

fn EGifBufferedOutput(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
    Buf: <mut<[]>, unknown, >,
    c: <i32, >,
) -> <i32, >;
```

##### More examples

```diff
fn DGifGetRecordType(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
    Type: <mut, i32, >,
) -> <i32, >;

fn DGifGetWord(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
    Word: <mut, i32, >,
) -> <i32, >;

fn DGifSetupDecompress(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <shared, struct(struct.GifFileType), >,
) -> <i32, >;
```

#### Weird function signatures

Often, Gordium infers `Rc` when `giblib-rs` uses `mut`. Sometimes, it infers `Rc<Cell>`, and once, it infers `ghost`.

```diff
/* mut -> ghost */

fn DGifGetCode(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <ghost, struct(struct.GifFileType), >,
    CodeSize: <mut, i32, >,
    CodeBlock: <mut, mut<[]>, unknown, >,
) -> <i32, >;


/* mut -> Rc */

fn DGifGetCodeNext(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
    CodeBlock: <mut, mut<[]>, unknown, >,
) -> <i32, >;

fn DGifGetExtension(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
    ExtCode: <mut, i32, >,
    Extension: <mut, mut<[]>, unknown, >,
) -> <i32, >;

fn DGifGetExtensionNext(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
    Extension: <mut, mut<[]>, unknown, >,
) -> <i32, >;


/* mut -> Rc<Cell> */

fn DGifSlurp(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc<Cell>, struct(struct.GifFileType), >,
) -> <i32, >;

fn EGifPutScreenDesc(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc<Cell>, struct(struct.GifFileType), >,
    Width: <i32, >,
    Height: <i32, >,
    ColorRes: <i32, >,
    BackGround: <i32, >,
    ColorMap: <shared, struct(struct.ColorMapObject), >,
) -> <i32, >;
```

Relatedly, `EGifOpenFileName` returns an `Rc` instead of a `Box`.

```diff
fn EGifOpenFileName(
    FileName: <shared, unknown, >,
    TestExistence: <i1, >,
    Error: <mut, i32, >,
-) -> <Box, struct(struct.GifFileType), >;
+) -> <Rc, struct(struct.GifFileType), >;
```

I think this means that it isn't possible to have a mutable reference to `GifFile`. I'm also not sure that there is a transformation from `Rc` to `Rc<Cell>` based on the paper and my reasoning.

##### More examples

```diff
/* mut -> Rc */

fn DGifGetLZCodes(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
    Code: <mut, i32, >,
) -> <i32, >;

fn DGifGetLine(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
    Line: <mut<[]>, i8, >,
    LineLen: <i32, >,
) -> <i32, >;

fn DGifGetPixel(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
    Pixel: <i8, >,
) -> <i32, >;
```

#### Less ownership when more makes more sense

I don't think this is a problem with correctness, mainly I think that `EGifCloseFile` _should_ take ownership of `GifFile` (so that it can't be used afterwards), but it doesn't.

```diff
fn EGifCloseFile(
-   GifFile: <Box, struct(struct.GifFileType), >,
+   GifFile: <mut, struct(struct.GifFileType), >,
    ErrorCode: <mut, i32, >,
) -> <i32, >;
```

### More "internally mutable" examples

```diff
 fn GifAllocColorArray(
     Count: <i64, >,
-) -> <Box<[]>, struct(struct.GifColorType), >;
+) -> <Rc<[Cell]>, struct(struct.GifColorType), >;

 fn GifAllocExtensionArray(
     Count: <i64, >,
-) -> <Box<[]>, struct(struct.ExtensionBlock), >;
+) -> <Rc<[]>, struct(struct.ExtensionBlock), >;
 
 fn GifAllocRaster(
     Count: <i64, >,
-) -> <Box<[]>, i8, >;
+) -> <Rc<[Cell]>, i8, >;

 fn GifReallocColorArray(
     Old: <shared<[]>, struct(struct.GifColorType), >,
     OldCount: <i64, >,
     Count: <i64, >,
-) -> <Box<[]>, struct(struct.GifColorType), >;
+) -> <Rc<[Cell]>, struct(struct.GifColorType), >;
 
 fn GifReallocExtensionArray(
     Old: <shared<[]>, struct(struct.ExtensionBlock), >,
     OldCount: <i64, >,
     Count: <i64, >,
-) -> <Box<[]>, struct(struct.ExtensionBlock), >;
+) -> <Rc<[]>, struct(struct.ExtensionBlock), >;
 
 fn GifReallocSavedImageArray(
     Old: <shared<[]>, struct(struct.SavedImage), >,
     OldCount: <i64, >,
     Count: <i64, >,
-) -> <Box<[]>, struct(struct.SavedImage), >;
+) -> <Rc<[Cell]>, struct(struct.SavedImage), >;
```

## 2026-10-21

> [!success]
> I  was able to successfully run the `giflib` translation on the OSC. The results seem promising, with not a lot of `unknown`s, as far as I can tell.

I'm not sure what the difference between `new_rust_sigs.txt` and `old_rust_sigs.txt` is, but they appear to be different, based on the output of `diff`.

> [!question]
> **Is `bool` interpreted as `i8`?**
>
> Yes, it appears that LLVM translates `bool` into `i8`, so we treat it as one as well.

Here are some of the differences:

```diff
struct struct.ColorMapObject (
    <ghost, i32, >,
    <ghost, i32, >,
-   <ghost, i8, >,
+   <ghost<Cell>, i8, >,
    <ghost, Rc<[Cell]>, struct(struct.GifColorType), >,
);
```

```diff
struct struct.ExtensionBlock (
-   <ghost, unknown, >,
+   <ghost<Cell>, unknown, >,
-   <ghost, Rc<[Cell]>, unknown, >,
+   <ghost<Cell>, Rc<[Cell]>, unknown, >,
-   <ghost, i32, >,
+   <ghost<Cell>, i32, >,
);
```

```diff
fn DGifGetLine(
-   GifFile: <mut, struct(struct.GifFileType), >,
+   GifFile: <Rc, struct(struct.GifFileType), >,
-   Line: <shared<[Cell]>, i8, >,
+   Line: <mut<[]>, i8, >,
    LineLen: <i32, >,
) -> <i32, >;
```

I'm not exactly sure how to analyze the results, if at all, so maybe I'll ask about that next meeting.

In the meantime, maybe I can get another library translated? Maybe I'll try doing `cmark` and `ziplib`.

### `cmark` Compilation

The first problem that I ran into was (like before) function pointers. However, in this case, function pointers are much more prevalent.

#### Function pointer removal
##### Removing the memory allocation indirection

In `cmark`, there is a `cmark_mem` struct that is passed around that contains the allocation functions for `calloc`, `realloc`, and `free`. I had AI inline the default memory allocation functions from `DEFAULT_MEM_ALLOCATOR` (`xcalloc`, `xrealloc`, and `free`).

##### Removing rendering callbacks

`cmark` uses callbacks in its `cmark_renderer` structure in order to allow for customization of how the output is rendered. To solve this, AI specialized the functions using macros. For more details, see `render.inc`.

##### The rest of the function pointers

Removing the rest of the function pointers basically involved specializing the code for the function. Nothing too difficult.

#### `ptrtoint` removal

The main strategy for this is to replace specific pointer subtraction (such as `curr - ` start) with an index/counter variable, since this tends to be the usecase.

#### `freeze` removal

This line in `commonmark.c` was producing a freeze instruction:

```c
(c == '`' || c == '<' || c == '>' || c == '\\' || cmark_isspace(c) ||
 c == ')' || c == '(')

```

Moving `cmark_isspace(c)` to the end fixed this.

```c
(c == '`' || c == '<' || c == '>' || c == '\\' ||
 c == ')' || c == '(' || cmark_isspace(c))
```

#### Allocation problems

Allocation with `calloc` and `realloc` was causing problems since it didn't have special handling like `malloc`. AI replaced these with specialized implementations of `calloc` and `realloc` that use `malloc` underneath.


#### `cmark_chunk`/`i64` conflict

I don't completely understand this one, but it has something to do with how LLVM treats `cmark_chunk` as a struct and an array of two `i64`s interchangeably.

```c
typedef struct {
  const unsigned char *data;
  bufsize_t len;
} cmark_chunk;
```

Somehow AI fixed this one, but I didn't completely understand the change. I'd have to go back and re-read it to understand.

#### Unions

The `as` union contains multiple different things that a node could be:

```c
union {
  cmark_list list;
  cmark_code code;
  cmark_heading heading;
  cmark_link link;
  cmark_custom custom;
  int html_block_type;
} as;
```

`refinator` doesn't know how to handle unions. Instead, we could represent `as` as a `struct`. It would waste space, but it would allow refinator to translate it.

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