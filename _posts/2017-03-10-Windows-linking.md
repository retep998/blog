---
layout: post
title: Linking libraries on Windows
published: false
---

One of the most common problems I see people face when using Rust on Windows is figuring out how to link libraries correctly. This blog post will serve as a guide to do things correctly.

## What is a library?

On Windows there are two kinds of libraries you'll be working with. Static libraries and import libraries both of which look like `foo.lib`. Both are passed to the linker the same way, so you might be wondering what the difference is? Well, when you link to an import library, your program will gain a runtime dependency on some DLL (or multiple DLLs) which might have a completely different name. Meanwhile, when you link to a static library, it does not directly result in any runtime dependencies on any DLLs.

You might be wondering whether you can just link to a DLL directly. No, no you cannot. If you try to pass a DLL to `link.exe` it will be very confused and have no idea what you're trying to do. MinGW theoretically can sometimes in certain situations link to DLLs directly, but it never worked when I tried it, so please do not rely on that behavior. MinGW even provides import libraries for all its DLLs because it doesn't trust linking to a DLL directly.

## Which kind do I use?

Let's go over the most important kinds and when you should use them. Kinds can be specified in multiple ways such as an attribute (`#[link(name = "foo", kind = "dylib")]`), an argument to rustc (`-l dylib=foo`), or build script output (`cargo:rustc-link-lib=dylib=foo`).

### static

Do not use this kind.

### dylib

This is the most versatile kind as it allows you to link basically anything. If you're targeting stable, this is currently your only sane choice, so don't bother with anything else.

### static-nobundle

This kind was recently added and is still unstable, so you can only use it if you use nightly. However it is useful when linking to a static library, because it ensures that Rust behaves correctly, both with regards to dllimport and dylibs.

## What crate type do I use to create a DLL?

`cdylib`. Never ever use `dylib` unless you fully understand what you're doing and you absolutely need its very special behavior, in which case you wouldn't be reading this guide. But because you are here, I'm telling you to just always use `cdylib`.

