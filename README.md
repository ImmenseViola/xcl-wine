# xcl-wine

An **unofficial fork** of [XCL](https://github.com/xcl-dev/xcl) (`xcl-dev/xcl`) that works on
getting XCL to run under **Wine / Proton on Linux**. It is not affiliated with the XCL project.
Please report problems with *this fork* here, not to upstream.

XCL is a compatibility layer that runs Xbox Series X|S (GDK) games you own on a PC. See
[`README.upstream.md`](README.upstream.md) for what XCL is and how it works.

## What this fork changes

On Windows, Microsoft's D3D12 accepts any object XCL hands it. Under Wine, vkd3d-proton checks that
objects carry its own vtable and refuses or crashes on the ones XCL has adopted. This fork puts the
native vtable back for the duration of those native calls:

- command allocators passed to list `Reset` / `CreateCommandList`
- the allocator's own `Reset`
- command lists passed to `ExecuteCommandLists`

## Status

Work in progress and experimental. Nothing here is a finished or supported way to run games.
Tested only on Ubuntu with GE-Proton, cross-built with `cargo xwin` for `x86_64-pc-windows-msvc`.

## Requirements and legal

You need your own legally obtained copy of any game. This repository contains no game files and
never will. XCL is licensed under **GPL-2.0**; this fork keeps the same license (see `LICENSE`)
and all upstream copyright notices.
