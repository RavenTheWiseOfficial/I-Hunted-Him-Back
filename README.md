# The Targeted Cyber Attack — Forensic Documentation

### TDF (The Digital Frontier) — Crypto Mining Startup

> **Disclaimer:** No malicious files are shared in this
> repository. All artifacts documented here are hex dumps,
> metadata extracts, registry data, and hash references only.
> The actual payloads will not be distributed under any
> circumstances.

-----

## Overview

This repository documents a real, targeted, multi-stage
cyberattack against a cryptocurrency mining startup —
TDF (The Digital Frontier) — that was in active development
in 2021.

The attack was identified and fully remediated through
**manual forensic analysis alone** — no drive wipe, no
professional IR team, no AV assist. Every payload was
found by hand.

-----

## Attack Summary

|Field                       |Detail                                            |
|----------------------------|--------------------------------------------------|
|**Target**                  |TDF (The Digital Frontier) — crypto mining startup|
|**Attack type**             |Targeted, multi-stage                             |
|**Initial compromise**      |September 2021                                    |
|**Payloads identified**     |5                                                 |
|**Peak AV detection rate**  |4/69 engines (Trojan)                             |
|**Other hashes**            |2/71 engines                                      |
|**Trojan confirmed**        |VirusTotal — April 19, 2026                       |
|**Webcam registry artifact**|August 15, 2025                                   |
|**Drive wipe required**     |No                                                |
|**Who found it**            |The victim. Manually. Alone.                      |

-----

## Attack Vector

Before a single payload was deployed, the attacker
identified:

- The target was building a **cryptocurrency mining startup**
- The target was active in the **Gala Games ecosystem**
- The target worked with **.eps and .ai design files** routinely
- Enough about the company’s **visual identity** to replicate
  it convincingly as a lure

**The lure:** Weaponized Adobe Illustrator EPS and AI files
placed in the target’s Downloads folder alongside real TDF
company files. Environmentally invisible by design.

**The trojan:** A weaponized Gala Games Launcher installer —
a platform the target actively used — making the download
contextually plausible.

-----

## Infection Chain

```
[DELIVERY]
├── Weaponized EPS/AI files impersonating TDF branding
└── Trojanized GalaLauncherSetup (v1.0.4 through v1.1.5)

[PERSISTENCE]
NSSM (Non-Sucking Service Manager)
Embedded unsigned inside trojanized installer
Registered as Windows auto-start service
Re-established 3x after removal attempts

[CONCEALMENT]
Two hidden services masquerading as Google Updater:
  GoogleUpdaterInternalService140.0.7272.0
  GoogleUpdaterService140.0.7272.0
Hidden at rootkit level
Verdict: HiddenService.Multi.Generic

[INJECTION]
Six inject.bin shellcode stubs
Timestamp: 2021-09-21 18:48:52 UTC — all six identical
Host: VSCode Python debugger (trusted signed process)
Living-off-the-land — not scrutinized by AV engines

[SUSTAINED ACCESS]
RAT infrastructure maintained
C2 traffic masked behind Google Updater paths
calls-wmi: hardware enumeration including camera/audio
```

-----

## Confirmed Artifacts

### 1. Malicious Installer Hashes

|File                       |SHA256                                                            |VT Detections|Tags                                                        |
|---------------------------|------------------------------------------------------------------|-------------|------------------------------------------------------------|
|GalaLauncherSetup-1.0.5.exe|`fddc9561fca3f353957ae73aa9be8848a8c699484d291a85538e0fbd303ae9b5`|4/69         |obfuscated, detect-debug-environment, long-sleeps, calls-wmi|
|GalaLauncherSetup-1.0.9.exe|`a94efb2fb59c54f90648a5652f2b5ebc1fdb55bab898c130cb828182ace5dc52`|2/71         |runtime-modules, direct-cpu-clock-access, overlay           |
|4259324125.exe             |`5be74fb52b5811981980ec5d5a122b21993af564dae3ad3cb6a45f1aaa7f432d`|2/71         |obfuscated, overlay, detect-debug-environment, long-sleeps  |

**Confirmed threat labels:**

- `HackTool.NSSM` — DrWeb, Rising
- `Trojan.Generic.TRFH1111` — QuickHeal
- `HackTool.NSSM!1.CABB (CLASSIC)` — Rising
- `Trojan.Win32.Genus.TGO` — VirIT

**Versioning significance:** Four installer versions across
several months. Each iteration added obfuscation and
anti-analysis capabilities. Active campaign maintenance —
not a static drop.

-----

### 2. Kaspersky Quarantine Metadata

```ini
[InfectedFile]
Src: C:\Program Files (x86)\Gala Launcher\util\nssm.exe
md5: D9EC6F3A3B2AC7CD5EEF07BD86E3EFBC
sha256: 472232CA821B5C2EF562AB07F53638BC2CC82EAE84CEA13FBE674D6022B6481C

[InfectedObject]
Type: Service | Name: Gala Launcher
Start: Auto (0x2)
Verdict: UnsignedFile.Multi.Generic

[InfectedObject]
Type: Service | Name: GoogleUpdaterInternalService140.0.7272.0
Suspicious states: Hidden service
Verdict: HiddenService.Multi.Generic

[InfectedObject]
Type: Service | Name: GoogleUpdaterService140.0.7272.0
Suspicious states: Hidden service
Verdict: HiddenService.Multi.Generic
```

-----

### 3. Inject.bin Shellcode Stubs

Six files. All 32 bytes. All identical UTC timestamp.
Simultaneous creation = automated dropper confirmed.

**Hex dump — all six files byte-for-byte identical:**

```
Offset  00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
000000  00 FF 00 FF 00 FF 00 FF 00 FF 00 FF 00 FF 00 FF
000010  00 FF 00 FF 00 FF 00 C3 17 00 08 00 16 00 17 00
```

`0xC3` at offset `0x17` = x86 RET instruction.
Classification: **shellcode trampoline stubs** — memory
redirect pointers used by the main injector.

**Confirmed injection host:**

```
VSCode Python debugger (debugpy)
ms-python.python-2021.9.x — attach_pid_injected
```

-----

### 4. Lure File — Weaponized Company Branding

**EPS binary header:**

```
Offset 0: C5 D0 D3 C6 — Windows Metafile/EPS magic bytes
```

**PostScript metadata:**

```
%%Creator: Adobe Illustrator(R) 24.1
%%CreationDate: 9/9/2021
%%BoundingBox: 0 0 1080 720
```

**Attribution artifact:** The `%%For` field in the
PostScript header reflects the registered username of
the Illustrator installation used to create the file.
The attacker did not sanitize this metadata before
delivery.

**Behavioral observation:** Opening these files caused
immediate severe system throttling consistent with
embedded execution logic. No AV engine flagged during
live execution.

-----

### 5. Registry Forensic Artifact — Webcam Access

```
HKCU\Software\Microsoft\Windows\CurrentVersion\
CapabilityAccessManager\ConsentStore\webcam\
NonPackaged\windows.immersivecontrolpanel_cw5n1h2txyewy
```

|Value              |Type     |Raw              |Decoded                |
|-------------------|---------|-----------------|-----------------------|
|LastUsedTimeStart  |REG_QWORD|0x1dc0dbfe8a1a0e2|2025-08-15 08:38:00 UTC|
|LastUsedTimeStop   |REG_QWORD|0x1dc0dbff2a64199|2025-08-15 08:38:17 UTC|
|LastUserAnnotated  |REG_DWORD|0x00000002       |Access granted         |
|PersistedInDatabase|REG_DWORD|0x00000001       |Persisted              |

**Session duration: 17 seconds.**

Identical subkey structure confirmed under:

```
ConsentStore\microphone\
  windows.immersivecontrolpanel_cw5n1h2txyewy
```

**Why this is anomalous:**

The subkey appears under **NonPackaged** — the registry
location reserved for raw Win32 executables operating
outside the app sandbox. A legitimate Settings app
entry appears under its named package key, not here.

The lowercase `windows` naming does not match any
Microsoft naming convention for this component.

The `calls-wmi` behavioral tag confirmed on the
trojanized installer is the mechanism for hardware
device enumeration including camera and audio devices.

-----

## Documented Timeline

|Date                           |Event                                                                                               |
|-------------------------------|----------------------------------------------------------------------------------------------------|
|**Sep 8–9, 2021**              |Lure EPS/AI files created. Attribution artifact embedded in metadata.                               |
|**Sep 2021**                   |Trojanized Gala Launcher deployed. NSSM persistence established.                                    |
|**Sep 21, 2021 — 18:48:52 UTC**|Six inject.bin stubs deployed simultaneously. VSCode Python debugger injection executed.            |
|**Oct 2021**                   |pip environment injection artifacts timestamped.                                                    |
|**Dec 2021**                   |Loader updated v1.0.4 → v1.0.5. Obfuscation added. Active iteration confirmed.                      |
|**Jul 22, 2025**               |Full remediation session. Gala Launcher purged. Registry cleaned. Services removed.                 |
|**Aug 15, 2025 — 08:38 UTC**   |Anomalous webcam registry artifact logged under NonPackaged. 17-second session. Analyst not present.|
|**Apr 19, 2026**               |Trojan hash confirmed via VirusTotal. Full forensic documentation compiled.                         |

-----

## Why the AV Ecosystem Failed

|Technique                 |Effect                                                                |
|--------------------------|----------------------------------------------------------------------|
|`detect-debug-environment`|Silent in sandbox analysis                                            |
|`long-sleeps`             |Outlasts sandbox timeout windows                                      |
|`obfuscated`              |Defeats signature detection                                           |
|`overlay`                 |Payload outside standard scan regions                                 |
|Living-off-the-land       |NSSM legitimate. Google Updater paths trusted. VSCode debugger signed.|
|Hidden services           |Invisible to standard enumeration APIs                                |
|`calls-wmi`               |Legitimate system API — not flagged                                   |
|Environmental lure        |Matched victim’s own workflow and branding                            |

**65–69 engines saw nothing.**
**Manual analysis found everything.**

-----

## Tools Used

|Tool                    |Role                                           |
|------------------------|-----------------------------------------------|
|VirusTotal              |Hash analysis, behavioral tags, vendor verdicts|
|TDSSKiller              |Rootkit/hidden service detection               |
|Windows Registry Editor |ConsentStore forensics, service enumeration    |
|PowerShell / CMD        |File system forensics, artifact recovery       |
|Hex analysis            |inject.bin byte-level content analysis         |
|Manual forensic analysis|Primary remediation vector                     |

-----

## Closing Note

Five payloads. One confirmed Trojan. Hidden rootkit
services. Process injection through a trusted debugger.
A lure file built by impersonating the victim’s own
company. Anomalous webcam registry access at 4:38 AM.
Months of sustained access masked behind Google
infrastructure.

The attacker had enough intelligence to build a targeted
campaign. They did not have enough discipline to clean
their metadata, sanitize their filenames, or clear their
Recycle Bin.

The lesson is not that antivirus is worthless. The lesson
is that antivirus is a floor, not a ceiling. Closing the
gaps requires a human who knows where to look.

-----

*Forensic documentation by Robert Boncimino*
*Security Architect | Threat Hunter*
*GitHub: RavenTheWiseOfficial*
*All malicious files remain secured and will not be shared.*
