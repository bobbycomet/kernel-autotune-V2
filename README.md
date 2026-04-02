# Kernel Autotune v2 is an intelligent system tuning utility designed to optimize Linux kernel parameters based on your specific hardware and running kernel. It provides predictable performance improvements by automatically detecting hardware and applying a tailored profile.

- Key FeaturesIntelligent Kernel Detection: Automatically identifies and tunes for XanMod, Liquorix, Zen, TKG, CK, Clear, and CachyOS kernels.Hardware Awareness: Detects if the system is a desktop, laptop, or handheld (like a Steam Deck or ROG Ally) to adjust power and performance balance. 

- Memory Management: Configures ZRAM and Zswap based on total RAM capacity.

- Optimizes swappiness, VFS cache pressure, and dirty ratios.

- Storage Optimization: Identifies SSD vs. HDD devices to set appropriate I/O schedulers (e.g., none for SSDs, bfq for HDDs).

- Enables fstrim.timer automatically for SSD maintenance.

- Network Performance: Enables BBR TCP congestion control and optimizes buffer sizes for high-speed networking.

- CPU Tuning: Sets the scaling governor based on the detected device type and power source (AC vs. Battery).

Get the deb here: https://github.com/bobbycomet/kernel-autotune-V2/releases/download/v2.0.0/kernel-autotune-pkg.deb

Once you have built the .deb package, install it using apt:
```
sudo apt install ./kernel-autotune.deb
```

## Usage

While the service runs automatically, you can interact with the script manually using the following commands:

```
kernel-autotune status
```

Manual Apply: Re-run the detection and apply optimizations immediately.

```
sudo kernel-autotune apply
```

View Logs: Check the history of applied changes.

```
tail -f /var/log/kernel-autotune.log
```

Configuration & State

- Config File: /etc/kernel-autotune/config.sh (Generated based on hardware detection).

- State File: /etc/kernel-autotune/state.json (Summary of the current active profile).

- Sysctl Drop-in: /etc/sysctl.d/99-kernel-autotune.conf.

Safety Features

- Deterministic Behavior: No runtime surprises; settings are validated against the current state.

- Thermal Aware: Adjusts Transparent Huge Pages (THP) on laptops to reduce heat generation.

- Fallback Chains: Uses safe defaults if specific kernel features or schedulers are missing.
