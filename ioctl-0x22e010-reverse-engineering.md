---
title: "Reverse Engineering IOCTL 0x22E010: The EDR-Killing Control Code"
date: 2026-04-14
updated: 2026-04-14
summary: "IOCTL 0x22E010 is a vendor-defined buffered I/O control code used by the Rentdrv2/PoisonX kernel driver to terminate arbitrary processes, including PPL-protected EDR services. It is associated with CVE-2023-44976 and has been weaponized in BYOVD attacks by the Agonizing Serpens (Agrius) APT group since October 2023."
---

# Reverse Engineering IOCTL 0x22E010: The EDR-Killing Control Code

## Executive Summary

IOCTL 0x22E010 is a kernel-level process termination control code exposed by the Rentdrv2 driver (Hangzhou Shunwang Technology). When called from kernel mode, it bypasses Windows Protected Process Light (PPL) to kill any process, including CrowdStrike Falcon and other EDR products. The driver carries a valid Microsoft signature and had zero antivirus detections at time of discovery.

While a vendor patch exists (versions after 2024-12-24), it does not mitigate the real-world threat. In BYOVD (Bring Your Own Vulnerable Driver) attacks, adversaries drop their own copy of the old, pre-patch signed binary onto the target system. Because the Microsoft signature remains valid, Windows loads it without complaint. The patch only fixes the vendor's distribution, not the attacker's copy. This technique has been weaponized in the wild since October 2023 by the Iranian APT group Agonizing Serpens (Agrius) against Israeli targets.

Effective mitigation requires Microsoft to revoke the driver's signature or add its hashes to the Windows Driver Blocklist (WDAC). Until then, organizations should enforce HVCI (Hypervisor-protected Code Integrity) and monitor for driver loads, the device path `\\.\{F8284233-48F4-4680-ADDD-F8284233}`, and IOCTL 0x22E010 calls.

---

## Current Status

- **IOCTL 0x22E010** is a process-termination control code exposed by the Rentdrv2/PoisonX Windows kernel driver
- Assigned **CVE-2023-44976** (CWE-782: Exposed IOCTL with Insufficient Access Control), CVSS 3.2 Low
- Exploited in the wild since **October 2023** by the Iranian-backed APT group Agonizing Serpens (Agrius) in attacks against Israeli higher education and tech sectors [Palo Alto Unit 42]
- Multiple public proof-of-concept tools exist (PoisonKiller, BadRentdrv2, PoisonX-Killer) [GitHub]
- The driver carries a valid Microsoft signature (signed 2025-03-25), with **0/71 VirusTotal detection rate** as of the PoisonKiller PoC publication [GitHub/Nekr0w]
- Vendor patch available: Rentdrv2 versions after 2024-12-24 address the vulnerability [NVD]

---

## IOCTL Bit Field Breakdown

The Windows `CTL_CODE` macro encodes four fields into a 32-bit IOCTL value:

```
CTL_CODE(DeviceType, Function, Method, Access)
```

### Decoding 0x0022E010

```
Hex:    0x0022E010
Binary: 0000 0000 0010 0010 1110 0000 0001 0000

Bits 31-16 (Device Type):  0x0022 = 34 = FILE_DEVICE_UNKNOWN
Bits 15-14 (Access):       0x3    = 3  = FILE_READ_ACCESS | FILE_WRITE_ACCESS
Bits 13-2  (Function Code): 0x804 = 2052 (vendor-defined, >= 0x800)
Bits 1-0   (Method):       0x0    = 0  = METHOD_BUFFERED
```

### Interpretation

| Field | Value | Meaning |
|-------|-------|---------|
| Device Type | `FILE_DEVICE_UNKNOWN` (0x0022) | Generic/unspecified device type, common for third-party drivers that don't fit a standard category |
| Required Access | `FILE_READ_ACCESS \| FILE_WRITE_ACCESS` (0x3) | Caller must have both read and write access to the device object |
| Function Code | 0x804 (2052) | Vendor-defined function. Microsoft reserves 0x000-0x7FF for standard functions; 0x800+ is for third-party use |
| Transfer Method | `METHOD_BUFFERED` (0x0) | Kernel copies input/output buffers via system buffer (Irp->AssociatedIrp.SystemBuffer), the safest transfer method |

### Reconstruction verification

```c
CTL_CODE(FILE_DEVICE_UNKNOWN, 0x804, METHOD_BUFFERED, FILE_READ_DATA | FILE_WRITE_DATA)
= (0x0022 << 16) | (0x3 << 14) | (0x804 << 2) | 0x0
= 0x00220000 | 0x0000C000 | 0x00002010 | 0x0
= 0x0022E010  // Confirmed
```

**WinDbg verification command:**
```
!ioctldecode 0x22E010
```

---

## The Driver: Rentdrv2 / PoisonX

### Identity

IOCTL 0x22E010 belongs to the **Rentdrv2** driver, developed by Hangzhou Shunwang Technology. The driver has also been referred to as **PoisonX.sys** in PoC tooling. There are reportedly 15+ variants sharing identical code [Medium/@jehadbudagga].

| Property | Value |
|----------|-------|
| Original Name | Rentdrv2.sys |
| PoC Name | PoisonX.sys |
| Vendor | Hangzhou Shunwang Technology |
| Signature | Microsoft Windows Hardware Compatibility Publisher |
| Device Path | `\\.\{F8284233-48F4-4680-ADDD-F8284233}` |
| VT Detection | 0/71 (at time of PoC publication) |

### What the driver does

Rentdrv2 is a legitimate internet cafe management driver used in China. Its intended purpose is to manage user sessions and system resources in rental computer environments. However, its IOCTL dispatch handler exposes a process termination function that operates at kernel level, bypassing all user-mode protections.

### The kill mechanism

When IOCTL 0x22E010 is received, the driver:

1. Reads the input buffer, which contains a PID as a decimal ASCII string
2. Calls `ZwOpenProcess` to obtain a kernel handle to the target process
3. Calls `ZwTerminateProcess` to kill the process

This is significant because `ZwOpenProcess` called from kernel mode bypasses Process Protection Light (PPL) restrictions. In user mode, `OpenProcess` against a PPL-protected process returns ACCESS_DENIED, but the kernel-mode `Zw*` functions operate with full privilege [Medium/@jehadbudagga].

### Input/Output format

```
Input:  PID as decimal ASCII string (e.g., "1234"), sent as bytes
Output: 16-byte buffer; returns "ok" on success
Method: METHOD_BUFFERED (via Irp->AssociatedIrp.SystemBuffer)
```

---

## Attack Pattern: BYOVD (Bring Your Own Vulnerable Driver)

### How it works

1. **Drop the driver**: Attacker places Rentdrv2.sys/PoisonX.sys on disk
2. **Load via service**: Create a kernel service (`sc create ... type=kernel binPath=...`) and start it
3. **Open device**: Call `CreateFileW` with device path `\\.\{F8284233-48F4-4680-ADDD-F8284233}`
4. **Send kill IOCTL**: Call `DeviceIoControl` with control code 0x22E010, passing the target PID as input
5. **EDR dies**: The kernel-level termination bypasses PPL, killing CrowdStrike Falcon, Defender, or any other EDR process

The driver loads without triggering code-integrity checks because it carries a valid Microsoft signature [GitHub/Nekr0w].

### Real-world usage

**Agonizing Serpens (Agrius)** - an Iranian-backed APT group - weaponized Rentdrv2 in attacks against Israeli higher education and technology sector organizations starting in October 2023. The group used a custom loader called `drvIX.exe` to communicate with the driver. Palo Alto's Unit 42 documented this as an upgrade in the group's capabilities, noting they had not used BYOVD techniques in previous campaigns [Palo Alto Unit 42].

---

## Reverse Engineering Methodology

### Tools for IOCTL analysis

| Tool | Purpose | Notes |
|------|---------|-------|
| **IDA Pro** | Static analysis/decompilation of the .sys driver | Primary tool used in the original RE analysis |
| **Ghidra** | Free alternative to IDA for driver decompilation | NSA's reverse engineering framework |
| **WinDbg** | Kernel debugging, `!ioctldecode` command | Built-in IOCTL decoder: `!ioctldecode 0x22E010` |
| **WinObj** (Sysinternals) | Discover device symbolic links at runtime | Used to find `\\.\{F8284233-48F4-4680-ADDD-F8284233}` |
| **OSR IOCTL Decoder** | Online tool to decode CTL_CODE fields | Quick bit field breakdown |
| **OSR Driver Loader** | Load/unload kernel drivers for testing | Used instead of `sc create` for lab environments |
| **IoctlHunter** | Dynamic IOCTL monitoring via DLL injection | Captures IOCTLs in a controlled lab environment |
| **VirusTotal** | Check driver signature and detection rate | Confirmed valid Microsoft signature |

### RE approach for this driver

The original reverse engineer (Jehad Abudagga) documented the following approach [Medium/@jehadbudagga]:

1. **Skip the DriverEntry**: The `DriverEntry` function was heavily obfuscated/mangled. Instead of fighting the obfuscation, the analyst went directly to the **IRP dispatch handler** (the function registered for `IRP_MJ_DEVICE_CONTROL`)
2. **Fix types and names**: IDA's initial decompilation output was difficult to read. Manual type annotation and variable renaming were required to produce clean pseudocode
3. **Identify the IOCTL dispatch table**: The dispatch handler compared incoming IOCTL codes and routed to different functions. Two IOCTLs were found; 0x22E010 was the one leading to the process kill function
4. **Trace the kill chain**: Following the 0x22E010 handler revealed the `ZwOpenProcess` to `ZwTerminateProcess` call sequence

### General IOCTL RE methodology

For reverse engineering any unknown IOCTL:

1. **Decode the bits**: Use `CTL_CODE` macro decomposition (as shown above) to understand device type, access requirements, function code, and transfer method
2. **Identify the driver**: Search for the IOCTL value in known databases (ioctls.net, the DDK headers, public malware/PoC repos)
3. **Load in disassembler**: Open the .sys file in IDA/Ghidra, locate `DriverEntry`, find the `IRP_MJ_DEVICE_CONTROL` dispatch function
4. **Find the switch/comparison**: The dispatch handler will compare `IoStackLocation->Parameters.DeviceIoControl.IoControlCode` against known values
5. **Trace the handler**: Follow the code path for your target IOCTL to understand what it does
6. **Dynamic analysis**: Use WinDbg kernel debugging to set breakpoints on `NtDeviceIoControlFile` and observe live IOCTL traffic

---

## Security Analysis

### CVE-2023-44976

| Field | Value |
|-------|-------|
| CVE ID | CVE-2023-44976 |
| CWE | CWE-782: Exposed IOCTL with Insufficient Access Control |
| CVSS 3.1 | 3.2 (Low) |
| Vector | AV:L/AC:L/PR:H/UI:N/S:C/C:N/I:N/A:L |
| Affected | Hangzhou Shunwang Rentdrv2 before 2024-12-24 |
| Exploited in Wild | October 2023 |
| NVD Published | August 1, 2025 |
| Fix | Update to Rentdrv2 versions released 2024-12-24 or later |

Source: [NVD CVE-2023-44976](https://nvd.nist.gov/vuln/detail/CVE-2023-44976), [Feedly CVE feed](https://feedly.com/cve/CVE-2023-44976)

**Note on CVSS score**: The 3.2 Low rating is arguably understated. While the attack vector is local and requires high privileges (admin to load a driver), the scope is changed (S:C) because it affects processes outside the vulnerable component's security context. The real-world impact of killing EDR processes to enable further attacks is substantially higher than the base score suggests.

### Threat landscape

- The vulnerability sat in **reserved CVE status for over a year** before publication (wild exploitation October 2023, NVD published August 2025) [Feedly]
- Multiple public PoC repositories exist on GitHub, lowering the barrier for copycat attacks
- The BYOVD technique using this driver is effective against any EDR that relies on PPL for process protection, not just CrowdStrike [GitHub/Muz1K1zuM]

### Detection opportunities

1. **Driver load monitoring**: Alert on PoisonX.sys, Rentdrv2.sys, or service creation for unfamiliar kernel drivers
2. **Device path monitoring**: Watch for `CreateFile` calls targeting `\\.\{F8284233-48F4-4680-ADDD-F8284233}`
3. **IOCTL monitoring**: Detect `DeviceIoControl` calls with code 0x22E010
4. **Driver hash blocklisting**: Add known Rentdrv2/PoisonX driver hashes to WDAC or HVCI blocklists
5. **Kernel callback routines**: Implement `PsSetCreateProcessNotifyRoutine` callbacks to detect unauthorized kernel-initiated process termination

---

## Confidence Assessment

| Claim | Confidence | Basis |
|-------|------------|-------|
| Bit field decoding of 0x22E010 | **High** | Mathematical decomposition per Microsoft CTL_CODE specification |
| Association with Rentdrv2/PoisonX driver | **High** | Multiple independent sources: CVE database, Unit 42 report, researcher blog, GitHub PoCs |
| Process kill mechanism (ZwOpenProcess/ZwTerminateProcess) | **High** | Confirmed by independent reverse engineering (Medium blog) and corroborated by PoC source code (GitHub) |
| CVE-2023-44976 details | **High** | Sourced directly from NVD |
| Agonizing Serpens attribution | **High** | Sourced from Palo Alto Unit 42 threat intelligence report |
| CVSS score being understated | **Medium** | Analytical judgment; the base score follows CVSS methodology but doesn't capture chained attack impact |
| 0/71 VT detection rate | **Medium** | Reported in PoisonKiller PoC README; may have changed since publication |

---

## Sources

### Primary Research

- Abudagga, Jehad. "Reverse Engineering a 0day used Against CrowdStrike EDR." Medium, April 2026. https://medium.com/@jehadbudagga/reverse-engineering-a-0day-used-against-crowdstrike-edr-a5ea1fbe3fd4
- "Researcher Reverse Engineered 0-Day Used to Disable CrowdStrike EDR." Cybersecurity News, April 14, 2026. https://cybersecuritynews.com/0-day-disable-crowdstrike-edr/

### Threat Intelligence

- Palo Alto Networks Unit 42. "Agonizing Serpens (Aka Agrius) Targeting the Israeli Higher Education and Tech Sectors." https://unit42.paloaltonetworks.com/agonizing-serpens-targets-israeli-tech-higher-ed-sectors/

### Vulnerability Databases

- NIST NVD. "CVE-2023-44976 Detail." https://nvd.nist.gov/vuln/detail/CVE-2023-44976
- Feedly. "CVE-2023-44976 - Exploits & Severity." https://feedly.com/cve/CVE-2023-44976
- CISA. "Vulnerability Summary for the Week of July 28, 2025." https://www.cisa.gov/news-events/bulletins/sb25-216

### Proof of Concept / Source Code

- BlackSnufkin/BYOVD - PoisonX-Killer (Rust implementation). https://github.com/BlackSnufkin/BYOVD/blob/main/PoisonX-Killer/src/main.rs
- Nekr0w/poisonkiller - PoisonKiller PoC. https://github.com/Nekr0w/poisonkiller
- j3h4ck/PoisonKiller - Fork/variant. https://github.com/j3h4ck/PoisonKiller
- Muz1K1zuM/PoisonKiller_bof - Cobalt Strike BOF version. https://github.com/Muz1K1zuM/PoisonKiller_bof

### Microsoft Documentation

- Microsoft Learn. "Defining I/O Control Codes." https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/defining-i-o-control-codes
- Microsoft Learn. "!ioctldecode." https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-ioctldecode
- Microsoft Learn. "Security Issues for I/O Control Codes." https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/security-issues-for-i-o-control-codes

### Tools

- OSR IOCTL Decoder (online). Referenced but not fetched.
- z4ksec. "IoctlHunter Release (v0.2)." https://z4ksec.github.io/posts/ioctlhunter-release-v0.2/
- daaximus. "Most IOCTLs mapped to their code names." GitHub Gist. https://gist.github.com/daaximus/e813aa52980fc2a97a8a8a1082338de4
