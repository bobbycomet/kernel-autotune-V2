# Kernel Autotune v2 - Security Policy

This document outlines the security considerations, threat model, and vulnerability reporting process for **Kernel Autotune v2**. As a solo dev, I do not have the funds for a bounty.

## 🛡️ Security Philosophy

Kernel Autotune runs at the highest privilege level (**root**/sudo) to modify critical system parameters.  
Our primary security goal is to ensure that performance optimizations **never** compromise system stability or introduce new attack vectors.

## 🔍 Security Measures

- **Root Enforcement**: The script strictly checks for `EUID == 0` before any installation or application of settings.
- **Safe Pathing**: All binaries and configuration files are installed in protected system directories (`/usr/local/bin`, `/etc/kernel-autotune`).
- **Input Validation & Hardening**: The script uses `set -euo pipefail` and `IFS=$'\n\t'` to fail fast on errors and handle input securely.
- **Predictable & Deterministic Configuration**: Settings are generated based on hardware detection rather than arbitrary user input.
- **Comprehensive Logging**: All actions are logged to the system journal (`journalctl`) and `/var/log/kernel-autotune.log` for auditability.

## 🛠️ Threat Model & Mitigations

| Threat                      | Mitigation Strategy |
|-----------------------------|---------------------|
| **Unauthorized Modification** | Binaries use `755` and configs use `644` permissions. Only root can modify installed files. |
| **System Instability**       | Safety fallbacks for sysctl parameters and I/O schedulers if the running kernel does not support them. |
| **Information Leakage**      | All state and log files are stored in root-owned directories with restrictive permissions. |
| **Persistence Hijacking**    | The systemd service uses a static `ExecStart` path pointing to `/usr/local/bin/kernel-autotune`. |

## ⚠️ Known Limitations

- **Privileged Execution**: This tool **requires root access** by design. Never run Kernel Autotune if you obtained it from an untrusted source.
- **SysRq Exposure**: The script enables specific `kernel.sysrq` values (176) to allow emergency sync and reboot functionality.

## ✉️ Reporting a Vulnerability

If you discover a security vulnerability in Kernel Autotune v2, please report it responsibly.

**Preferred contact method:**

- Join our Discord server: [https://discord.gg/fMCpeNCxhv](https://discord.gg/fMCpeNCxhv)
- Message **@maintainer** (or ping in the `#security` or `#support` channel if available)

**Reporting Guidelines:**

1. Do **not** open a public GitHub issue.
2. Provide a clear description of the vulnerability, including:
   - Steps to reproduce
   - Potential impact
   - Any suggested mitigation (if known)
3. We aim to acknowledge your report within **48 hours** and provide a resolution or mitigation plan within **7 days**.

---

Thank you for helping keep Kernel Autotune secure!  
Responsible disclosure is greatly appreciated.
