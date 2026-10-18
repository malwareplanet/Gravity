# Gravity — by Malware Planet

A single PowerShell script that sets up a complete, working malware-analysis environment on Windows 11 — 28 tools across 10 categories, auto-resolved dependencies, and safety checks that refuse to run destructively on a physical machine.

**Website:** https://www.malwareplanet.com · **Author:** Neeraj Rawat ([Securix](https://www.malwareplanet.com))

---

## ⚠️ Authorized Use Only

This toolkit is built for use inside a dedicated, isolated malware-analysis VM — nowhere else. By running it, you take full responsibility for the environment you run it in.

**"Isolated" means the whole access chain, not just the VM.** A VM with no internet access and a clean snapshot is not meaningfully isolated if you reach it from a machine that *is* on your corporate network — via RDP, console access, shared folders, or clipboard sync. Malware that achieves any form of breakout (hypervisor escape, RDP session compromise, clipboard/drive-redirection abuse) can use that access path to pivot from the "isolated" VM back into your real network through the device you used to reach it.

If you're setting this up in a corporate environment:
- Reach the analysis VM from a **dedicated, non-domain-joined physical machine**, not your everyday corporate laptop
- Disable clipboard sync, drive redirection, and shared folders on the hypervisor/RDP session
- Treat the VM as hostile even when "contained" — assume breakout is possible, not impossible
- Get explicit written authorization from your organization before running live samples anywhere on corporate-connected infrastructure

This script disables Windows Defender, the firewall, and Windows Update **by default, and only when it detects it is running on a virtual machine**. It refuses to do so on what it believes is physical hardware (see [Safety behavior](#safety-behavior) below). It is provided with no warranty. You are solely responsible for the environment, network, and samples you use it with.

---

## Quick start

Don't pipe this straight from the internet into execution — review it first, the same way you'd want any script that disables Defender and the firewall to be reviewed before it runs.

```powershell
# 1. Download
Invoke-WebRequest -Uri https://github.com/malwareplanet/Gravity/releases/latest/download/Install-MalwarePlanet-Gravity.ps1 -OutFile Install-MalwarePlanet-Gravity.ps1

# 2. (Optional but encouraged) verify the checksum against the one in the release notes
Get-FileHash .\Install-MalwarePlanet-Gravity.ps1 -Algorithm SHA256

# 3. Unblock and run, elevated (Run as Administrator)
Unblock-File .\Install-MalwarePlanet-Gravity.ps1
.\Install-MalwarePlanet-Gravity.ps1
```

## Safety behavior

Before disabling any defenses, Gravity checks several independent signals (hypervisor-present flag, BIOS/model manufacturer strings, VM guest-tools services) to confirm it's actually running on a virtual machine:

- **Looks like a VM** → installs the toolkit and disables Defender real-time/cloud protection, the firewall (all profiles), and Windows Update, as expected for an isolated analysis environment.
- **Looks like physical hardware** → installs the toolkit **only**. All protections are left on, and a warning is printed. Nothing destructive happens by accident.

Tamper Protection is **never** touched either way — Microsoft removed the registry/PowerShell path for that years ago. Turn it off manually first (Windows Security → Virus & threat protection → Manage settings → Tamper Protection → Off) if you want Defender fully disabled on a VM.

## Parameters

| Flag | What it does |
|---|---|
| `-ToolsRoot <path>` | Where tools are installed. Default `C:\MalwarePlanet\Tools` |
| `-KeepDefenses` | Skip disabling Defender/firewall/Windows Update entirely — install tools only |
| `-ForceDisableDefensesOnHost` | Override the physical-hardware refusal above. Only use this if you are certain the detection is wrong for your specific virtualization setup |
| `-IncludeDefenderRemover` | Downloads (never runs) a third-party Defender removal tool, for reference. Off by default |
| `-AddDefenderExclusion` | Adds the tools folder to Defender exclusions before installing, so downloads aren't quarantined mid-run |
| `-SkipDependencies` | Skip installing runtimes (Java, .NET, Python, VC++ Redist, Graphviz) |
| `-SkipShortcuts` | Don't create desktop shortcuts |
| `-OpenManualPages` | Opens the vendor pages for any manual-only tools in your browser at the end |

## What it installs

**28 tools across 10 categories** — pulled directly from the script, always accurate to what you're running:

| # | Category | Tools |
|---|---|---|
| 1 | Static & PE Analysis | CFF Explorer, PE-bear, PE Studio, Detect It Easy, AnalyzePESig, TrID / TrIDNet, ExifTool |
| 2 | Disassembly & Debugging | x64dbg, dnSpy, Cutter |
| 3 | Decompilation & RE | Ghidra, ILSpy, Bytecode Viewer, JADX |
| 4 | Packer & Shellcode | scDbg, Speakeasy |
| 5 | Dynamic & Behavioral | Process Hacker / System Informer, Regshot, ProcDOT, Sysinternals Suite |
| 6 | Hex & Binary Diffing | Resource Hacker, Meld |
| 7 | Hashing & Utilities | HashMyFiles, 7-Zip |
| 8 | Network Analysis | Wireshark, FakeNet-NG |
| 9 | String Extraction | FLOSS |
| 10 | Signature Matching | YARA |

Plus 6 supporting prerequisites installed automatically: VC++ Redistributable 2015–2022, .NET 8 Desktop Runtime, Java JRE/JDK 25 (Temurin), Python 3.14, and Graphviz.

Each tool is sourced in this order, first success wins: **winget → latest GitHub release → direct vendor download → manual (if none of the above exist)**. Only **Regshot** is manual-only in the current release — its upstream project has never published GitHub release binaries or a winget package.

## Known limitations

- **Tamper Protection** can't be scripted off — manual toggle only (see above)
- **Regshot** stays manual-only — no scriptable source exists upstream
- **CFF Explorer** downloads NTCore's full ExplorerSuite installer, which bundles several tools together and needs to be run once to install — it doesn't get an automatic desktop shortcut for this reason, since the installer places the real apps outside the tools folder with no reliable way to locate them back afterward. The downloaded installer itself is always in the tools folder (printed at the end of every run) ready to run

## Troubleshooting

**`winget` fails on every single package with the same error, pointing to `aka.ms/winget-command-install`**
This means `winget.exe` on this machine is a non-functional stub (common on freshly provisioned VMs that never had the Store activated). Gravity detects this automatically and attempts a repair by installing the real App Installer package directly — if that repair also fails, every tool still falls back to direct downloads, so the run continues regardless.

**A Microsoft Store/`msstore` terms prompt appears**
Every install is pinned to the `winget` community source specifically to avoid this. If you still see it, you're likely running an older copy of the script — re-download the latest release.

**Defender shows as "disabled" but Windows Security still shows it on**
This is Tamper Protection silently reverting the change — Gravity verifies the real state afterward and will print `STILL ON` with instructions if this happens, rather than falsely claiming success.

**The ".NET Framework 3.5" step seems to hang**
It's capped at 90 seconds and will time out safely if Windows Update doesn't respond — the rest of the script continues either way. TrIDNet (the GUI) will prompt for this itself on first launch if it wasn't enabled automatically.

## Verifying your download

```powershell
Get-FileHash .\Install-MalwarePlanet-Gravity.ps1 -Algorithm SHA256
```
Compare the output against the SHA256 hash published in the corresponding [GitHub Release](../../releases) notes.

## License

GNU GPL v3.0 — see [LICENSE](LICENSE). You're free to use, modify, and redistribute this, including commercially — the one condition is that anyone who distributes a modified version must also release their changes under GPL v3, so improvements stay open for everyone.

"Malware Planet" is a registered trademark of Neeraj Rawat / Malware Planet. "Gravity" is the product name used under that brand. The license covers the code; it does not grant rights to the Malware Planet name or branding — forks must use different branding.

## Found a security issue?

Please don't open a public issue for it. Email **admin@malwareplanet.com** first so it can be addressed responsibly before public disclosure.
