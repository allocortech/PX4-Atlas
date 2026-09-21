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

## Building

Build in the PX4 dev container that upstream uses for this release, so the
binaries reproduce:

```
podman run --rm -v "$PWD:$PWD" -w "$PWD" \
  docker.io/px4io/px4-dev-nuttx-focal:2022-08-12 \
  bash -lc 'git config --global --add safe.directory "*"; \
            make allocortech_atlas-v6x_default && \
            make allocortech_atlas-v6x_bootloader'
```

`px4io/px4-dev-nuttx-focal:2022-08-12` is the image upstream PX4
`release/1.16` builds NuttX targets in (`.github/workflows/checks.yml`); docker
works in place of podman. Mount the tree at the same path inside the container
as outside, because absolute paths end up in the build state. Initialise both
NuttX submodules first (`nuttx` and `apps`), and do not carry host-built
helpers from `platforms/nuttx/NuttX/nuttx/tools` into the container.

A build with a different toolchain produces different bytes and a different
size. The app is close to its flash ceiling, so check the reported FLASH usage
after any change.

The bootloader build rewrites `extras/allocortech_atlas-v6x_bootloader.bin` in
the working tree; commit that file when the bootloader is meant to change, and
restore it otherwise so the committed binary never drifts from the sources
beside it.

PX4 embeds `git describe` output, so the same sources built from a tag and
from a branch differ by that string alone (about ten bytes).
When changing the NuttX pin, push the selected NuttX commit first so fresh
clones and CI can retrieve it, then commit the PX4 submodule update.

The cleaned firmware needs a hardware acceptance run before deployment.
Earlier instrumented builds validated nonempty file integrity and short
benchmarks in both widths. The known FAT empty-file read defect remains
separate. Sustained logging, physical power-cycle persistence, and additional
boards remain to be qualified.
