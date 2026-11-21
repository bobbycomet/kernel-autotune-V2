# Changelog

## 2.4.0

- Added distribution-family, init-system, kernel memory-model, cgroup,
  power-manager, ZRAM-manager, PSI, and container detection.
- Added safer CPU tuning and external ZRAM policies for mixed distro and
  init-system environments.
- Added a canonical `zram-impl.sh` implementation for more reliable ZRAM
  setup and version-matched upgrades.
- Updated the systemd unit and package metadata for 2.4.0.
- Improved read-only and fallback behavior for modern Linux systems.
