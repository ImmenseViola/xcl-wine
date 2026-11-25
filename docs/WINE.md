# Running XCL under Wine / Proton (Linux) - notes

Unofficial and experimental. Tested on Ubuntu 26.04, GE-Proton, Mesa/RADV (AMD RX 9070).

## Build (cross-compile from Linux)
    cargo xwin build --release --target x86_64-pc-windows-msvc
    cp target/x86_64-pc-windows-msvc/release/xcl.exe ~/xcl-run/xcl.exe

## Run
    export WINEPREFIX="$HOME/xcl-run/prefix/pfx"
    GEP=$(ls -d ~/GE-Proton* | tail -1)
    cd ~/xcl-run
    XCL_NETWORK=0 WINEDLLOVERRIDES="d3d12,d3d12core,dxgi=n,b" WINEDEBUG=-all \
      "$GEP/files/bin/wine" ./xcl.exe run "Z:\\home\\$USER\\game" > run.out 2>&1
Add `XCL_D3D12_TRACE=1` to log every D3D12 call (very slow, huge log).

## Symbolizing a crash (xcl.exe+0x... frames)
    cd target/x86_64-pc-windows-msvc/release
    llvm-symbolizer --obj=xcl.exe --relative-address --inlines 0x759dc4 0x392eb6

## The core problem and fix
vkd3d-proton checks that objects passed to it carry its own vtable; XCL adopted objects carry an
extended one. `real::with_native_vtable(s)` swaps the native vtable back for the native call.
Fixed so far: allocator in list Reset / CreateCommandList, allocator Reset, lists in
ExecuteCommandLists. Other adopted objects (PSOs, resources, fences, heaps) may need the same.

## Progress log
1. Crash: "Invalid command allocator" (allocator vtable) -> fixed.
2. Crash in d3d12core from ExecuteCommandLists (list vtable) -> fixed.
3. Now: game reaches shader compilation, then crashes with a null write in the game's own code
   (game+0x2215c35) after repeated `err:winevulkan:signaller_worker wait timed out` lines.
