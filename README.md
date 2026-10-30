# Turnip for Xiaomi Mi 8 (dipper) / Adreno 630

Builds Mesa Turnip for the Mi 8 with the KGSL kernel interface.

The workflow produces two variants:

| Variant | Vulkan reported | Use |
|---|---|---|
| `vk13-stable` | 1.3.363 | Games and normal daily use; upstream behaviour |
| `vk14-experimental` | 1.4.363 | HyperOS 4 glass rendering; experimental |

## Why two builds?

Upstream Turnip intentionally caps A6xx (including Adreno 630) at Vulkan 1.3
because not every Vulkan 1.4 limit is met. HyperOS 4's glass renderer needs a
driver that advertises 1.4, so the experimental variant patches only the A630
physical-device version.

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

   If `/vendor` is read-only, use your existing overlay/module mechanism.
   Do not replace the whole vendor partition.

3. Keep:

   ```properties
   ro.hardware.vulkan=turnip
   ro.hwui.use_vulkan=true
   ```

4. Reboot.

Do not enable the old `Adreno-Gpu-Driver` module at the same time. Only one
Vulkan driver should be active.

## Notes

- Kernel: 4.9 / KGSL
- GPU: Adreno 630
- Target: Xiaomi Mi 8 (`dipper`)
- Mesa source: pinned in the workflow (`MESA_REF`)
- The Android feature XML may still report Vulkan 1.1; the driver's physical
  device version is what matters for the renderer.
