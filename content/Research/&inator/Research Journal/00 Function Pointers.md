## 2026-09-30 (Wednesday, Sept 30)

I need to add functions to the `STypeFrag` enum, and I'm considering two different variations.

The first is just to store a type variable in the function.

```rust
STypeFrag::Fn(FnTypeVar)
```

This would be similar to a struct type fragment `STypeFrag::Struct(String)`, with the "identity" being a type variable instead of a name.

However, this would be hard to get working with the `From<T: Deref<Target = Type>>` implementation, since there wouldn't be any way to get a fresh type variable. In addition, there wouldn't be a way to retain information about the param types and return types from the LLVM type.

So instead, I think it's better to represent it like this:

```rust
STypeFrag::Fn {
    param_types: Vec<STypeFrag>,
    return_type: STypeFrag,
}
```

However, I'm not sure if this makes sense, since this doesn't exactly preserve the types. For example, for the return type, it only preserves the _first_ `STypeFrag`, not any of the following. Maybe this is okay, I guess I just don't understand exactly how the `STypeFrag` is being used. I'll leave this as is for now, but I might need to switch it up later.

---

While working on testing function pointer conversion, I ran into this error :

```
thread 'main' (44604) panicked at src/lang/program.rs:71:50:
`name` is a global variable
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

This happens in `Labels::extract_var` in the following lines of code:

```rust
llvm_ir::Operand::ConstantOperand(c) => match &**c {
    llvm_ir::Constant::GlobalReference { name, ty: _ } => {
        Some(*self.globals.get(name).expect("`name` is a global variable"))
    }
    _ => None,
},
```

Following it up the stack leads to the branch of `Instruction::from_instr` that handles `llvm_ir::Instruction::Callee`. 

Specifically, when it is getting the argument variables, it somehow fails to extract some of the argument `VarIdent`s. My guess is that it is having a hard time with passing in `add`, which is a global constant? I'll dig in a bit more to find out.

```rust
let arg_vars: Vec<Option<VarIdent>> = llvm_instr
    .arguments
    .iter()
    .map(|(operand, _)| labels.extract_var(func, operand))
    //                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .collect();
```

---

My new guess is that functions aren't declared in the global scope as a "variable", so they don't have a variable ID.

I need to figure out how globals are defined.

---

I tried adding a global variable ID for each function, but that led to needing to some how instantiate a global variable for each function in a way that didn't make a ton of sense.

My next plan is to try and store a `VarIdent` alongside each function.

- [ ] Find a way to assign a `VarIdent` to each function

## 2026-09-25 (Friday, Sept 25)

I need to **translate C function pointers into Rust function pointers**. My original plan was to translate this:

```c
int calc(int (*f)(int*, int*), int x, int y) {
    return (*f)(&x, &y);
}
```

into this:

```rust
fn calc(f: impl Fn(&i32, &i32) -> i32, x: i32, y: i32) -> i32 {
    ...
}
```

So,

```c
int (*f)(int*, int*)
```

becomes

```rust
impl Fn(&i32, &i32) -> i32
```

However, I realized that **this can't be stored in a struct**. I considered wrapping it in a `Box`:

```rust
Box<dyn Fn(&i32, &i32) -> i32>
```

That would make it possible to store it in a struct without using generics. However, **it wouldn't be possible to store it in a constant**. My next idea, which seemed obvious in retrospect, was to use **Rust's function pointers**. Something like

```rust
fn(&i32, &i32) -> i32
```

This would make a copyable constant pointer that would be useable at runtime. In addition, it would match the semantics of the C program more closely, since it would imply that there are no closures being passed around, only static function pointers.

### Conflicting function inferred types

There's one problem with these ideas, though. **This assumes that all functions have the same type inferred.** However, it's possible, for example, that `add` might be inferred as a `fn(&i32, &i32) -> i32` while `mult` might be inferred as a `fn(Rc<i32>, Rc<i32>) -> i32`. I can think of two ways to resolve this.

#### Insert pointer transformations

The first is to **create a closure that inserts pointer transformations**. This means that if `add` and `mult` need to be passed into the same place, we would do something like the following:

```rust
fn add(a: &i32, b: &i32) -> i32 { /* ... */ }
fn mult(a: Rc<i32>, b: Rc<i32>) -> i32 { /* ... */ }
fn calc(f: fn(Rc<i32>, Rc<i32>) -> i32, a: i32, b: i32) -> i32 { /* ... */ }

let foo = calc(mult, 1, 2);
let bar = calc(|a, b| add(&*a, &*b), 1, 2);
```

However, we would run into an issue: **introducing pointer transformations for functions would require closures, which are not static function pointers**. Technically, both would satisfy an `impl Fn(Rc<i32>, Rc<i32>) -> i32` type, but I think this would add a lot of **complexity** with figuring out pointer transformations, determining what type everything should be, and so on. I think there's a simpler way.

#### Make all function types the same

When we are doing type inference, we could **require that all functions that are passed into this have the same type inferred**. So, both `add` and `mult` need to be either `fn(&i32, &i32) -> i32` or `fn(Rc<i32>, Rc<i32>) -> i32`. **If they match type signatures, then they can always be used in place of each other.**

```rust
fn add(a: Rc<i32>, b: Rc<i32>) -> i32 { /* ... */ }
fn mult(a: Rc<i32>, b: Rc<i32>) -> i32 { /* ... */ }
fn calc(f: fn(Rc<i32>, Rc<i32>) -> i32, a: i32, b: i32) -> i32 { /* ... */ }

let foo = calc(mult, 1, 2);
let bar = calc(add, 1, 2);
```

 I think that it would make sense to make all function types the same, although I don't yet have any real-world evidence for this claim. My intuition is that **functions that are used in this way will be expected to have the same signature anyways**, since that is how it is in the original code.

Now, I should acknowledge that **I haven't tried implementing this in the code yet**, so this may be a more complex solution than I expect. But the idea seems relatively simple.

### Conclusion

To translate C function pointers to Rust, I think the best path forward would be to **use Rust's function pointers** and to add a requirement that **all of the pointers that are used for a given function parameter or struct field must have the same type**. As far as I can think of, this would most accurately reflect the semantics of the original program. However, there may be better solutions that I haven't considered yet, so I'd welcome any critiques.

---

I'm trying to figure out how to do type inference for function pointers, and I think this should work.

Each time a local value is called as a function, that local value receives the type of `fn(i)`, where `i` is a unique number. Anytime a function is assigned to a value, that value also gets a type of `fn(i)`, where `i` is a unique number. We also note that the constant function has a type that equals `fn(i)`. Then, we use equivalence to find all of the functions that should have equivalent types, and we ensure that those functions share the exact same type in our constraints.

We will probably have to have a map that tracks the "phantom function types".

### TODO

- [ ] Work on implementing function pointer type inference in the code