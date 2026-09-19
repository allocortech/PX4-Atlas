# Allocortech Atlas v1.16 integration

The company `atlas/release-1.16` branch starts from the Atlas board port in
[PX4 #28756](https://github.com/PX4/PX4-Autopilot/pull/28756), commit
`116ef1e096e9bef9356383ac60d752cb18da0290`.

Its NuttX submodule uses [allocortech/NuttX](https://github.com/allocortech/NuttX).
The gitlink pins an exact commit; the branch setting in `.gitmodules` is
maintenance metadata, not a floating build dependency.

The selected NuttX changes fix EXT_CSD DMA alignment and add opt-in four-bit
MMC support for STM32H7. Atlas enables `CONFIG_MMCSD_MMC_WIDEBUS` and disables
the two one-bit-only settings. The configured MMC clock divider is unchanged.
For one-bit operation, disable that option and restore
`CONFIG_SDIO_WIDTH_D1_ONLY=y` and `CONFIG_SDMMC2_WIDTH_D1_ONLY=y`.

Build with `make allocortech_atlas-v6x_default` after initializing submodules.
When changing the NuttX pin, push the selected NuttX commit first so fresh
clones and CI can retrieve it, then commit the PX4 submodule update.

The cleaned firmware needs a hardware acceptance run before deployment.
Earlier instrumented builds validated nonempty file integrity and short
benchmarks in both widths. The known FAT empty-file read defect remains
separate. Sustained logging, physical power-cycle persistence, and additional
boards remain to be qualified.
