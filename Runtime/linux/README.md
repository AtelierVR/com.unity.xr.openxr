# UnityOpenXR.so — Linux native core plugin

This directory is reserved for the Linux build of Unity's proprietary
`UnityOpenXR` native core plugin.

## What lands here

- `Runtime/linux/UnityOpenXR.so` — the core OpenXR <-> Unity bridge plugin
  (exported entry points such as `main_LoadOpenXRLibrary`, `session_*`,
  `NativeConfig_*`, and all `xr*` API wrappers).
- `RuntimeLoaders/linux/libopenxr_loader.so` — the Khronos OpenXR loader.
  This file IS built from open source (see below) and is already present.

## Why UnityOpenXR.so is not built here

The core `UnityOpenXR` native plugin is **Unity proprietary** and its C++
source lives in Unity's internal `xr.sdk.openxr` repository (see
`package.json` -> `repository.url`). Unity only ships pre-built binaries
for Windows (`Runtime/windows/x64/UnityOpenXR.dll`), macOS
(`Runtime/osx/UnityOpenXR.dylib`) and UWP — **not** Linux. The source is not
publicly available, so it cannot be compiled from this repository.

The shared runtime-loading glue that the core plugin uses to dynamically
load the OpenXR loader (`Shared/Native~/plugin_load.cpp`) already has full
POSIX/Linux `dlopen` support (`XR_USE_PLATFORM_LINUX`), so the core plugin
does compile and link for Linux when built inside Unity — the absence is
purely a distribution gap.

## How to obtain the Linux core plugin

1. Build it against a Unity Editor installation that includes the OpenXR
   native sources (Unity employees with access to `xr.sdk.openxr`).
2. Or, take the official Windows `.dll`/macOS `.dylib` and rebuild the core
   from those (not feasible without source).
3. Contact Unity to request a Linux Standalone build of
   `UnityOpenXR.so`.

Once a `Runtime/linux/UnityOpenXR.so` is available, drop it here; the
`Runtime/linux/UnityOpenXR.so.meta` PluginImporter is already configured to
enable it for `Standalone: Linux64` and the Linux Editor (x86_64), and
`OpenXRLoader` already advertises `StandaloneLinux64` support.

## Rebuilding libopenxr_loader.so from source

```bash
git clone --depth 1 --branch release-1.1.54 https://github.com/KhronosGroup/OpenXR-SDK.git
cd OpenXR-SDK
CC=clang CXX=clang++ cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_TESTS=OFF \
  -DBUILD_SHARED_LIBS=ON \
  -DBUILD_LOADER=ON \
  -DBUILD_API_LAYERS=OFF \
  -DBUILD_WITH_SYSTEM_JSONCPP=OFF \
  -DBUILD_LOADER_WITH_EXCEPTION_HANDLING=OFF \
  -DCMAKE_CXX_FLAGS="-DJSON_USE_EXCEPTIONS=0"
cmake --build build --target openxr_loader -j"$(nproc)"
cp build/src/loader/libopenxr_loader.so.1.1.54 \
   Packages/com.unity.xr.openxr/RuntimeLoaders/linux/libopenxr_loader.so
```

Notes:
- `BUILD_WITH_SYSTEM_JSONCPP=OFF` vendors jsoncpp so the `.so` is
  self-contained (no external `libjsoncpp` dependency).
- `-DJSON_USE_EXCEPTIONS=0` satisfies the `-Wundef` check when exception
  handling is disabled in the vendored jsoncpp build.
