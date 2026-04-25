<p align="center">
  <img src="https://github.com/bobbycomet/kernel-autotune-V2/blob/main/kernelautotune.png?raw=true" width="50%">
</p>

# Kernel Autotune v2.2.0

TLDR: Makes your system more predictable by automatically tuning your Linux kernel, memory, CPU, storage, and networking, no manual tweaking required.

An intelligent Linux kernel tuning utility that detects your hardware and running kernel, then applies a tailored performance profile automatically. Designed for Ubuntu users coming from Windows who want better system performance without digging through forums or terminals. Best paired with: [Process Sentry](https://github.com/bobbycomet/Process-Sentry), automatic background process control and cleanup.

## What it changes

- CPU governor (performance/schedutil)
- ZRAM/swap behavior
- I/O scheduler (SSD vs HDD)
- Network congestion control (BBR/cubic)
- Transparent Huge Pages
- Sysctl performance tuning

## What improves?

- More consistent performance under load
- Fewer slowdowns from memory pressure
- Better responsiveness on lower-RAM systems
- Improved battery behavior on laptops

## Is this safe?

Yes. Kernel Autotune:
- Uses only standard sysctl and kernel interfaces
- Does not patch or recompile your kernel
- Can be fully reverted with one command
- Fully reversible with `kernel-autotune uninstall`

## Key Features

**Kernel Detection**
Automatically identifies XanMod, Liquorix, Zen, TKG, CK, Clear, CachyOS, and generic kernels. Each kernel gets its own tuning defaults. For example, XanMod gets the `performance` CPU governor and lower swappiness, while Liquorix/Zen/CachyOS use `schedutil` with a higher ZRAM ratio.

**Hardware Awareness**
Detects desktops, laptops, and handhelds (Steam Deck LCD/OLED, ASUS ROG Ally, Lenovo Legion Go) using DMI product name rather than hostname, so detection is reliable regardless of what you've named your machine. Laptops and handhelds automatically switch to `schedutil` when running on battery.

**Memory Management**
Configures ZRAM compression swap scaled to your total RAM:
- ≤2 GB RAM — ZRAM at 150% of RAM, swappiness 30
- ≤4 GB RAM — ZRAM at 100% of RAM, swappiness 25
- ≤8 GB RAM — ZRAM at 75% of RAM, Zswap also enabled
- ≤16 GB RAM — ZRAM at 50% of RAM
- >16 GB RAM — ZRAM at 30% of RAM, swappiness 5

Handhelds use `lz4` compression and ZRAM at 100% of RAM for fast, low-power swap. Laptops also use `lz4`. Desktops default to `zstd`. ZRAM and Zswap are mutually exclusive; enabling both is blocked to prevent double-compression. Dirty writeback timings (`dirty_expire_centisecs`/`dirty_writeback_centisecs`) are derived from `DIRTY_RATIO` automatically for internal consistency.

**Transparent Huge Pages (THP)**
- XanMod — `madvise`
- Generic kernel — `defer+madvise`
- Laptops with a thermal sensor — `defer+madvise`
- Handhelds — `never` (reduces heat)

**Storage Optimization**
Scans every non-loop, non-zram, non-optical, non-device-mapper block device and sets the appropriate I/O scheduler: `none` for SSDs/NVMe (bypasses the scheduler for fast storage), `bfq` for HDDs. Scheduler availability is checked before applying; unsupported schedulers are silently skipped. Enables `fstrim.timer` automatically when any SSD is detected.

**Network Performance**
Probes `/proc/sys/net/ipv4/tcp_available_congestion_control` before committing to BBR. If BBR is available, it's used with the `fq` qdisc; if not, it falls back to `cubic` with `fq_codel`. Also sets TCP fast open, MTU probing, window scaling, and large socket buffer sizes (16 MB rmem/wmem).

**CPU Governor**
Set per kernel type (XanMod/TKG/CK/Clear default to `performance`; others default to `schedutil`). For laptops and handhelds, the governor is overridden to `schedutil` automatically when on battery, checked at apply time against multiple known AC supply paths.

**NUMA Support**
Detects multi-node NUMA systems and enables `kernel.numa_balancing` with a lower `vfs_cache_pressure` (50). On single-node systems, `numa_balancing` is disabled.

**Additional sysctl settings applied**
- `kernel.sysrq = 16` (sync only)
- `kernel.panic = 10`/`kernel.panic_on_oops = 1`
- `vm.overcommit_memory = 0`
- `fs.file-max = 2097152`
- `fs.inotify.max_user_watches = 524288`/`max_user_instances = 1024`
- `net.core.somaxconn = 8192`/`netdev_max_backlog = 16384`
- `net.ipv4.tcp_slow_start_after_idle = 0`

## Installation

Download the script and install it:

```
sudo bash kernel-autotune install
```

This copies the script to `/usr/local/bin/kernel-autotune`, runs hardware detection, applies settings immediately, and installs a systemd oneshot service (`kernel-autotune.service`) that re-applies settings on every boot.

If you prefer the `.deb` package, use your package installer (recommended) or use sudo:

```
wget https://github.com/bobbycomet/kernel-autotune-V2/releases/download/v2.2.0/kernel-autotune-2.2.0.deb
sudo apt install ./kernel-autotune-2.2.0.deb
```

## ZRAM Note (Important)

Kernel Autotune manages ZRAM itself and will stop the default `zram.service` to avoid conflicts.
In rare cases, if swap behavior seems incorrect, you can disable the system service manually:

```
sudo systemctl stop zram.service
sudo systemctl disable zram.service
```
Running both can cause conflicts, so the service is automatically stopped to prevent race conditions.

## Usage

```
kernel-autotune [--dry-run] [--quiet|--verbose] {install|apply|regen-config|uninstall|status|--version}
```

**Apply settings now** (respects any edits you've made to the config):
```
sudo kernel-autotune apply
```

**Preview what would change without touching anything** (no root required):
```
kernel-autotune --dry-run apply
```

**Regenerate config from hardware detection** (backs up the existing config first):
```
sudo kernel-autotune regen-config
```

**Show current state, active ZRAM, and recent logs:**
```
kernel-autotune status
```

**View the full log:**
```
tail -f /var/log/kernel-autotune.log
```

**Uninstall everything** (including logs, config, sysctl drop-in, and the systemd service):
```
sudo kernel-autotune uninstall
```

### Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--dry-run` | `-n` | Print what would be done; make no changes. No root required. |
| `--quiet` | `-q` | Suppress informational output; warnings and errors still shown. |
| `--verbose` | `-v` | Show per-device and per-step detail. |
| `--version` | `-V` | Print version and exit. |
| `--help` | `-h` | Show usage. |

## Configuration & State

| Path | Purpose |
|------|---------|
| `/etc/kernel-autotune/config.sh` | Generated from hardware detection. **Edit freely** — changes persist across reboots and `apply` runs. Run `regen-config` to reset to detected defaults (existing config is backed up automatically). |
| `/etc/kernel-autotune/state.json` | JSON summary of the last apply run: detected hardware, all config values, which steps succeeded, and which (if any) failed. |
| `/etc/sysctl.d/99-kernel-autotune.conf` | Sysctl drop-in written on each apply. |
| `/usr/local/bin/kernel-autotune-zram.sh` | Generated ZRAM helper script with parameters baked in. |
| `/var/log/kernel-autotune.log` | Timestamped log of every apply run. |

## Config Validation & Auto-Migration

Before applying, the script validates `config.sh` for syntax errors, missing required keys, and out-of-range values (e.g., `SWAPPINESS` must be 0–200, `THP` must be one of `always`, `madvise`, `defer+madvise`, or `never`). If hardware-detection keys (like `HAS_THERMAL` or `IS_NUMA`) are missing from an older config, they are re-detected and appended automatically rather than causing a hard failure.

## Safety Features

- **Idempotent ZRAM**: Skips teardown and rebuild if active ZRAM devices already match the configured size and count.
- **Concurrency lock**: An exclusive flock prevents two simultaneous invocations from racing on config, or sysfs writes.
- **Config sanitization**: Values sourced from hardware detection are stripped to safe characters before being written to the shell config file.
- **Partial-failure tracking**: Each tuning step (sysctl, zram, zswap, io_scheduler, cpu_governor, thp, fstrim) is tracked independently. Failures are reported in the `status` output without aborting the remaining steps.
- **Fallback chains**: BBR falls back to cubic; z3fold zpool falls back to zbud; unsupported I/O schedulers are skipped per-device.
- **Dry-run mode**: Fully safe, prints every action that would be taken without writing anything to disk or sysfs.
