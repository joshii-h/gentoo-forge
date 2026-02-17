# gentoo-forge

Optimized Gentoo Linux configuration for a desktop/gaming workstation running OpenRC + KDE Plasma (Wayland) + AMD GPU. This combination is rarely documented end-to-end, so this repo serves as a public reference.

## System Overview

| Component | Details |
|-----------|---------|
| **CPU** | Intel Core i7-11700F (8C/16T) |
| **GPU** | AMD Radeon RX 7900 XT/XTX (Navi 31, amdgpu) |
| **RAM** | 32 GB DDR4 |
| **Storage** | Samsung 980 PRO NVMe (btrfs) + SATA drives |
| **Init** | OpenRC + elogind (no systemd) |
| **Desktop** | KDE Plasma 6 on Wayland |
| **Audio** | PipeWire + WirePlumber |
| **Display Manager** | SDDM |

## Key Features

- **System-wide LTO** (`-flto=16 -fdevirtualize-at-ltrans -fno-semantic-interposition`) with a curated exception list for packages that break with LTO
- **ccache** (10 GB) + **tmpfs** portage build directory for fast compilation
- **Native Steam** with the full `abi_x86_32` dependency chain (70+ packages)
- **OpenRC + elogind** - no systemd, with polkit wheel rule for privilege escalation
- **AMD GPU** (amdgpu/radeonsi) with Vulkan, Wayland, and ROCm SMI support
- **Firefox from source** with PGO, hardware acceleration, and PipeWire integration
- **Kernel/sysctl tuning** for desktop responsiveness (swappiness, cache pressure, dirty ratios)
- **btrfs** with subvolumes (@, @home, @snapshots), zstd compression, and async discard
- **User patches** via `/etc/portage/patches/` (e.g., libX11 XKB crash fix)

## File Overview

```
etc/portage/
  make.conf                    # CFLAGS, LTO, ccache, parallel builds, GPU, USE flags
  package.env                  # Per-package LTO/optimization exceptions
  package.use/
    steam                      # Full abi_x86_32 chain for native Steam (~70 packages)
    firefox                    # PGO, hardware accel, PipeWire, xvfb for PGO builds
    abi32-debuginfod            # 32-bit deps for elfutils debuginfod (KDE crash handler)
    discover                   # Flatpak backend for KDE Discover
    grub                       # GRUB mount support
    installkernel              # Dracut initramfs generation
    podman                     # nftables backend for iptables
    xorg-drivers               # Disable radeonsi DDX (use modesetting instead)
  package.accept_keywords/
    steam                      # steam-overlay + libudev-compat + game-device-udev-rules
    gamemode                   # ~amd64 for Feral GameMode
    podman-compose             # ~amd64
    rocm-smi                   # ~amd64 for AMD GPU monitoring
    vanilla-sources            # ~amd64 for latest kernel sources
  env/
    no-lto.conf                # Disable LTO for incompatible packages
    no-devirtlto.conf          # Disable -fdevirtualize-at-ltrans
    no-seminterpos.conf        # Re-enable semantic interposition
  patches/
    x11-libs/libX11/           # XKB crash fix (MR#293) - applied automatically by portage
  repos.conf/
    gentoo.conf                # Main Gentoo repo with GPG verification
    eselect-repo.conf          # Steam overlay

etc/polkit-1/rules.d/
  49-wheel.rules               # Allow wheel group for polkit admin actions (OpenRC essential)

etc/sysctl.d/
  99-desktop-tuning.conf       # VM tuning for desktop responsiveness

etc/sddm.conf.d/
  01gentoo.conf                # Disable virtual keyboard
  10-keyboard.conf             # Point to Xsetup script

etc/sddm/
  Xsetup                       # Set keyboard layout for SDDM login screen

etc/conf.d/
  keymaps                      # Console keyboard layout (Swiss German)

etc/env.d/
  02locale                     # System locale (en_US.UTF-8 with Swiss regional formats)

var/cache/ccache/
  ccache.conf                  # ccache settings (10G, compression, sloppiness)

fstab.example                  # Reference fstab with btrfs subvolumes + tmpfs portage
```

## Highlights

### LTO Exception List

`make.conf` enables aggressive LTO flags globally. `package.env` maps 35+ packages to `no-lto.conf` that are known to break with LTO (glibc, mesa, gcc, llvm, python, rust, openssl, ...). Additional per-flag exceptions exist for `-fdevirtualize-at-ltrans` and `-fno-semantic-interposition`.

### Steam abi_x86_32 Dependency Chain

Native Steam on Gentoo requires explicit `abi_x86_32` USE flags on ~70 packages across multiple dependency chains (Mesa, fontconfig, udev, vulkan-loader, audio libs). The `package.use/steam` file documents the full chain with comments explaining each group.

### libX11 XKB Crash Patch

`patches/x11-libs/libX11/mr293-fix-xkb-crash.patch` fixes a crash in `XkbRefreshKeyboardMapping` when multiple keyboard devices are present. Applied automatically by Portage's user patch system.

### OpenRC + Polkit

Without systemd, the `49-wheel.rules` polkit rule is essential for allowing the `wheel` group to perform administrative actions through KDE's privilege escalation dialogs.

## Usage

> **These are personal configuration files.** They must be adapted to your system.

Things you will need to adjust:

- **`make.conf`**: `MAKEOPTS` and `-flto=` thread count to match your CPU
- **`fstab.example`**: Replace `<UUID-*>` placeholders with your actual UUIDs (`blkid`)
- **`package.use/steam`**: You may need fewer or more `abi_x86_32` entries depending on your `@world`
- **Keyboard layout**: `keymaps`, `Xsetup`, and locale files assume Swiss German - change to yours
- **`VIDEO_CARDS`**: Change if you don't have an AMD GPU

## Disclaimer

This is a personal configuration reference, not a turnkey solution. Configurations are specific to this hardware and use case. No warranty. Use at your own risk. Always review changes before applying them to your system.

## License

[GPL-3.0](LICENSE)
