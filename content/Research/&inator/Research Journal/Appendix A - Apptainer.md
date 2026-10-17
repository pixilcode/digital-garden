> [!quote]
> Apptainer is a _container_ platform. It allows you to create and run containers that package up pieces of software in a way that is portable and reproducible.

So apptainer is like docker? It seems like it uses docker behind the scenes.

> [!question]
> **When I run `apptainer shell rust-ubuntu.sif`, what environment am I placed in?**
> 
> When you are inside the shell spawned by `apptainer`, you are **the same user as you are in the host system**.
> 
> You can **run programs inside a container as if they were running on your host system**.

> [!info]
> In Docker terminology, the _Singularity Image File_ (SIF) is a _container_, not an _image_. In other words, a `.sif` file contains the _state_ of a system, not instructions on how to _build_ that system.

After reading, it looks like I need to bind `&inator` when building the `.sif` file? I can do this using [the `--bind` option](https://apptainer.org/docs/user/latest/build_a_container.html#bind).h

---

I tried this by running:

```sh
apptainer build --bind ~/Projects/refinator/:/refinator refinator.sif ~/Projects/refinator/rust-ubuntu.def
```

But it gives the following error:

```plaintext
WARNING: While bind mounting '/home/osu/Projects/refinator:/refinator': destination is already in the mount point list
WARNING: By using --writable, Apptainer can't create /refinator destination automatically without overlay or underlay
FATAL:   container creation failed: mount hook function failure: mount /home/osu/Projects/refinator->/refinator error: while mounting /home/osu/Projects/refinator: destination /refinator doesn't exist in container
FATAL:   While performing build: while running engine: while running %post section: exit status 255
```

---

Next, I tried adding a setup section to `rust-ubuntu.def` to create the needed directory. I also added an instruction to the post-setup section to copy the project from the bind point to `/opt/refinator`.

```
%setup
    mkdir -p $APPTAINER_ROOTFS/mnt/refinator

%post
    ...
    
    # Copy the project from the mount point `/mnt/refinator` to `/opt/refinator`
    cp -r /mnt/refinator /opt/refinator
```

This appears to have worked.

> [!question]
> I wonder if it would be better to just build refinator, then get rid of rust and refinator code?

Running

```sh
apptainer build --bind ~/Projects/refinator/:/mnt/refinator refinator.sif ~/Projects/refinator/rust-ubuntu.def
```

produces a `refinator.sif` file. Running that `refinator.sif` with

```sh
apptainer shell refinator.sif
```

produces the expected container.

However, when I navigate to `/opt/refinator` and attempt to run it with

```sh
cargo run -- benchmarks/amp
```

it eventually errors with

```plaintext
error: failed to open: /opt/refinator/target/debug/.cargo-build-lock

Caused by:
  Read-only file system (os error 30)
```

I think this is why refinator wasn't included in the image.

---

My new working theory is that the instructions in refinator's `README-OSC.md` assume that refinator is already in `~/refinator`.

---

My theory is right. In the instructions, it indicates that each time that refinator changes, you need to rsync _refinator_, not the _`rust-ubuntu.sif` image_.