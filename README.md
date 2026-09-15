# linux-ttm-test

Experimental Arch Linux kernel package carrying the one-line TTM
swapout fix for the hibernation/resume GPU crash reported as
[drm/amd#5387](https://gitlab.freedesktop.org/drm/amd/-/issues/5387).

Upstream: Vadim Nikitushkin's `drm/ttm: fix swapped-out resources never
leaving their bulk_move range`
([Sept 9, 2026](https://lists.openwall.net/linux-kernel/2026/09/09/2805)),
`Reviewed-by: Thomas Hellström + Christian König`, queued in
`drm-misc-fixes` (`3db7d7d58341` plus fix-up, see
[Sept 10 thread](https://lists.openwall.net/linux-kernel/2026/09/10/2267)),
`Cc: stable # v7.1+`.

This replaces the earlier Thomas Hellström `[CI v2] drm/ttm: Represent
LRU bulk moves as nested sublists` test patch (+214/-183). Per
Hellström's own Sept 10 review, that series only *appeared* to fix the
issue while keeping the resource on the bulk sublist; the one-line
`ret > 0` fix addresses the actual root cause (a `b2ed01e7ad3d`
regression that skipped bulk-move removal on every successful
`ttm_tt_swapout()`), so the heavy rewrite was dropped. Do not use
Samuel Ainsworth's withdrawn v1 patch either; it was reported to
introduce another use-after-free.

This is a stopgap until the fix lands in a stable release and stock
Arch. Keep a stock or LTS kernel installed and bootable.

## Patch identity

`ttm-swapout-bulk-move-fix.patch` changes one line in
`drivers/gpu/drm/ttm/ttm_bo.c` (`ttm_bo_swapout_cb()`):

```diff
-		if (!ret) {
+		if (ret > 0) {
```

SHA-256:

```text
ddda3065bee9d410ec1255fde79ca552b1a3d95e899aa157df37409db19ded73
```

## Build

Install the Arch build dependencies, then build the kernel and headers:

```sh
MAKEFLAGS=-j16 makepkg -s
```

Sixteen jobs completed successfully on a 32 GiB host. Use fewer jobs on a
memory-constrained system.

Install both packages together so external modules can build against the test
kernel:

```sh
sudo pacman -U linux-ttm-test-*.pkg.tar.zst linux-ttm-test-headers-*.pkg.tar.zst
```

The package installs alongside stock Arch kernels. Boot-loader entries,
initramfs generation, Secure Boot signing, and hibernation parameters remain
machine-specific and are intentionally not modified by this repository.
