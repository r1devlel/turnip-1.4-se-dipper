# Turnip for Xiaomi Mi 8 (dipper) / Adreno 630

Builds Mesa Turnip for the Mi 8 with the KGSL kernel interface, targeting the
current HyperOS 4 / Android 17 (API 37) userspace.

The workflow produces two variants:

| Variant | Vulkan reported | Use |
|---|---|---|
| `os-v14` | 1.4.363 | HyperOS 4 glass rendering; Android 17 / API 37; experimental |
| `vk13-stable` | 1.3.363 | Games and normal daily use; upstream behaviour |

## Build profile

- Mesa `main` pinned at `e04357c464296e6b0a0bda612d5b2fb5deb6fa82`
  (Mesa `27.0.0-devel`, 2026-11-09)
- Android NDK r30 (`30.0.16248370`)
- Android 17 target (`ANDROID_PLATFORM=37`, `-Dplatform-sdk-version=37`)
- Vulkan-only: `-Dvulkan-drivers=freedreno`, `-Dgallium-drivers=`
- KGSL-only: `-Dfreedreno-kmds=kgsl`
- No OpenGL/EGL/GLX/GBM/LLVM/tools/tests
- Release build, stripped, SONAME `vulkan.turnip.so`
- On-disk shader cache enabled
- LTO is not used because Mesa explicitly rejects LTO builds

The SDK package `platforms;android-37` is not required for this C/C++ build;
the NDK r30 toolchain supplies the API 37 sysroot and compiler wrappers.

## Why two builds?

Upstream Turnip intentionally caps A6xx (including Adreno 630) at Vulkan 1.3
because not every Vulkan 1.4 limit is met. HyperOS 4 glass needs a driver that
advertises 1.4, so `os-v14` patches only the A630 physical-device version.

The 1.4 build is not a magic performance upgrade. Use the 1.3 build for games
unless a game specifically needs 1.4. Keep both and test per use case.

## Install

1. Back up the current driver:

   ```sh
   cp /vendor/lib64/hw/vulkan.turnip.so \
      /sdcard/vulkan.turnip.so.backup
   ```

2. Replace it with the driver from the chosen archive:

   ```sh
   # after extracting the archive
   cp vendor/lib64/hw/vulkan.turnip.so /vendor/lib64/hw/vulkan.turnip.so
   chmod 644 /vendor/lib64/hw/vulkan.turnip.so
   ```

   If `/vendor` is read-only, repack/flash the vendor image through your normal
   mechanism. Do not replace the whole vendor partition blindly.

3. Keep:

   ```properties
   ro.hardware.vulkan=turnip
   ro.hwui.use_vulkan=true
   ```

4. Reboot.

Do not enable an old Adreno driver module at the same time. Only one Vulkan
driver should be active.

## Notes

- Kernel: 4.9 / KGSL
- GPU: Adreno 630
- Target: Xiaomi Mi 8 (`dipper`)
- The Android HAL entry point is exported as `HMI` (`hwvulkan_module_t`);
  Android's Vulkan loader does not look for the desktop `vk_icd*` exports.
- The Android feature XML may still report Vulkan 1.1; the driver's physical
  device version is what matters for the renderer.
