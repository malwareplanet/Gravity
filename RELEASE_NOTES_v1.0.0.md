# Gravity v1.0.0 — by Malware Planet

First public release.

## What it does

One PowerShell script sets up a complete malware-analysis environment on Windows 11: 28 tools across 10 categories, 6 supporting runtimes, automatic dependency resolution, and a VM-safety check that refuses to disable system defenses on what it detects as physical hardware.

## Highlights

- **28 analysis tools, 10 categories** — static/PE analysis, disassembly & debugging, decompilation, packer/shellcode, dynamic & behavioral analysis, hex/binary diffing, hashing utilities, network analysis, string extraction, and signature matching (YARA)
- **Three-tier source fallback per tool** — winget → latest GitHub release → direct vendor download, so a single broken source never blocks the whole run
- **Automatic VM detection** before touching Defender, the firewall, or Windows Update — refuses on physical hardware by default
- **Self-healing winget** — detects and repairs a non-functional winget stub automatically, common on freshly provisioned VMs
- **Honest verification, not blind trust** — actually checks Defender's real state after attempting to disable it, since Tamper Protection can silently revert changes with no error returned
- **Desktop shortcuts** for every installed tool, regardless of install method (portable download, winget, or CLI-only)

## Installation

```powershell
Invoke-WebRequest -Uri https://github.com/malwareplanet/Gravity/releases/download/v1.0.0/Install-MalwarePlanet-Gravity.ps1 -OutFile Install-MalwarePlanet-Gravity.ps1
Unblock-File .\Install-MalwarePlanet-Gravity.ps1
.\Install-MalwarePlanet-Gravity.ps1
```

See the [README](../../blob/main/README.md) for full documentation, parameters, and the authorized-use notice.

## Verify this download

```
SHA256: 9a866b3df945c1cde64ee6b5f4d43da22af609c53d533f2ce57ac51f6810f7ff
```
```powershell
Get-FileHash .\Install-MalwarePlanet-Gravity.ps1 -Algorithm SHA256
```

## Known limitations in this release

- Tamper Protection cannot be disabled by any script — manual toggle only, by design on Microsoft's part
- Regshot is manual-download only; no scriptable source exists upstream
- A small number of direct-installer tools (e.g. CFF Explorer) may not get an automatic desktop shortcut if their installer places the real app outside the tools folder — the tool itself still installs correctly, and the full tools folder path is always printed at the end of the run

## Requirements

- Windows 11 (Home, Pro, or Enterprise)
- Elevated PowerShell (Run as Administrator)
- Internet access

---

Full tool list, parameters, troubleshooting, and the authorized-use disclaimer: see [README.md](../../blob/main/README.md).
