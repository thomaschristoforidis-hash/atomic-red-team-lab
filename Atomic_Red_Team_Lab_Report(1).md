# Atomic Red Team Lab Report

**Detection Testing & ATT&CK Technique Emulation**

| Field | Details |
|-------|---------|
| **Author** | Thomas Christoforidis |
| **Role Target** | Junior SOC Analyst / Cyber Security Analyst |
| **Date** | August 2026 |
| **Tool** | Atomic Red Team + Invoke-AtomicRedTeam |
| **Focus** | Controlled technique emulation and detection validation |

---

## 1. Executive Summary

Practical use of Atomic Red Team to emulate selected MITRE ATT&CK techniques. Tests executed on a Windows endpoint with Sysmon; telemetry reviewed and detection visibility assessed.

---

## 2. Evidence – Installation

```powershell
Install-AtomicRedTeam -getAtomics
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```
Verified with `Invoke-AtomicTest T1082 -ShowDetailsBrief`.

---

## 3. Evidence – Techniques Executed & Telemetry

| ATT&CK ID | Technique | Test Focus | Sysmon ID 1 Observed | CommandLine Sample |
|-----------|-----------|------------|----------------------|--------------------|
| T1082 | System Information Discovery | systeminfo | Yes | systeminfo |
| T1033 | System Owner/User Discovery | whoami | Yes | whoami /all |
| T1087 | Account Discovery | net user | Yes | net user |
| T1016 | System Network Config Discovery | ipconfig | Yes | ipconfig /all |
| T1059.001 | PowerShell | Basic / encoded | Yes | powershell.exe -enc ... |

---

## 4. Evidence – Sample Sysmon Event from ART Test (T1033)

| Field | Value |
|-------|-------|
| TimeCreated | 2026-08-03 15:01:44 |
| Image | C:\Windows\System32\whoami.exe |
| CommandLine | whoami /all |
| ParentImage | C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe |
| User | LAB\asmith |
| Computer | WKS01.lab.local |

**Reconstruction:** Technique fully reconstructable from Sysmon alone. Parent-child relationship clear.

---

## 5. Evidence – Detection Validation Results

| Technique | Telemetry Visible? | Existing SIEM Alert? | Status |
|-----------|--------------------|----------------------|--------|
| T1082 | Yes (Sysmon 1) | No dedicated alert | Partial |
| T1033 | Yes (Sysmon 1 + 4688) | No dedicated alert | Partial |
| T1059.001 | Yes (Sysmon 1 + 4104) | Custom rule tested | Detected (after custom rule) |
| T1087 | Yes (Sysmon 1) | No dedicated alert | Partial |

**Key finding:** Strong telemetry exists for discovery techniques; formal alerting was the main gap — addressed for PowerShell via custom Wazuh rule.

---

## 6. Cleanup Evidence

```powershell
Invoke-AtomicTest T1082 -Cleanup
Invoke-AtomicTest T1033 -Cleanup
```
Cleanup commands executed after each test where available. VM snapshot used as additional safety control.

---

## 7. Conclusion

Atomic Red Team successfully used for controlled ATT&CK emulation. Evidence recorded for process creation events, detection validation status, and gap analysis.

---

**End of Report**
