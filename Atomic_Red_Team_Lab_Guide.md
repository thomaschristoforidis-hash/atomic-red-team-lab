# Atomic Red Team Home Lab Guide

**Detection Testing & ATT&CK Technique Emulation for SOC Analysts**  
Version 1.0 | August 2026

---

## 1. Overview & Objectives

This guide covers the practical use of **Atomic Red Team** — an open-source library of simple, focused tests mapped to MITRE ATT&CK techniques. It is one of the best ways for SOC analysts and detection engineers to generate realistic attacker activity in a controlled lab and then validate whether their detections (Sysmon, Wazuh, Event Logs, etc.) fire correctly.

**Primary Goals:**
- Install and run Atomic Red Team on a Windows lab machine
- Execute selected atomic tests safely
- Observe the resulting telemetry in Sysmon, Windows Event Logs, and Wazuh
- Map activity to MITRE ATT&CK techniques
- Document detection coverage and gaps
- Produce portfolio-ready evidence of detection testing

**Why this matters for SOC roles:**
- Understanding how attacker techniques appear in logs is essential for writing and tuning detections
- Atomic Red Team provides a repeatable, low-risk way to generate that activity
- Demonstrates proactive detection engineering skills valued by Australian employers

---

## 2. Lab Architecture (Recommended)

Use your existing lab components:

```
+------------------+     Atomic Tests      +------------------+
|  Windows Client  | --------------------> |  Sysmon +        |
|  (WKS01)         |                       |  Windows Event   |
|  + Atomic Red    |                       |  Logs            |
|    Team          |                       +--------+---------+
+------------------+                                |
                                                    | Forwarded
                                                    v
                                           +------------------+
                                           |  Wazuh SIEM      |
                                           |  (Detection &    |
                                           |   Alerting)      |
                                           +------------------+
```

- Run Atomic Red Team on the domain-joined Windows client (or a standalone Windows VM)
- Capture telemetry with Sysmon + Security Event Logs
- Forward to Wazuh for detection validation
- Optional: capture network traffic with Wireshark

---

## 3. Installation

### 3.1 Prerequisites
- Windows 10/11 lab machine (preferably domain-joined)
- PowerShell (Administrator)
- Internet access for initial download (or offline copy of the repo)
- Sysmon already installed (strongly recommended)

### 3.2 Install Atomic Red Team (Invoke-AtomicRedTeam)

Open PowerShell **as Administrator**:

```powershell
# Install the execution framework
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics

# Import the module (add to profile if desired)
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

### 3.3 Verify Installation

```powershell
# List available techniques
Invoke-AtomicTest T1003 -ShowDetailsBrief

# Check prerequisites for a specific test
Invoke-AtomicTest T1059.001 -CheckPrereqs
```

---

## 4. Safe Testing Principles

- **Only run in an isolated lab** — never on production or personal systems
- Start with low-impact tests (discovery, defence evasion that only touches files/processes)
- Prefer tests that do not require elevated privileges when possible
- Snapshot your VM before running more aggressive techniques
- Clean up after tests (`-Cleanup` parameter where available)
- Document everything you run

---

## 5. Recommended First Tests (Beginner-Friendly)

| ATT&CK Technique | Atomic Test Focus | What to look for in logs |
|------------------|-------------------|--------------------------|
| **T1082** – System Information Discovery | systeminfo, hostname, etc. | Process Creation (Sysmon 1 / 4688) |
| **T1059.001** – PowerShell | Simple PowerShell commands | Sysmon 1, PowerShell logs, Script Block |
| **T1033** – System Owner/User Discovery | whoami, query user | Sysmon 1, Security 4688 |
| **T1087** – Account Discovery | net user, net localgroup | Sysmon 1, command-line visibility |
| **T1016** – System Network Configuration Discovery | ipconfig, route print | Sysmon 1 |
| **T1049** – System Network Connections Discovery | netstat | Sysmon 1 |
| **T1518** – Software Discovery | registry / wmic style checks | Sysmon 1 |
| **T1003** (careful) | Credential dumping related | Sysmon 10 (Process Access), etc. — use only in isolated lab |

Start with discovery techniques — they are low risk and produce clear process-creation telemetry.

---

## 6. Execution Workflow

### Step-by-step for each test

1. **Choose a technique**
   ```powershell
   Invoke-AtomicTest T1082 -ShowDetails
   ```

2. **Check prerequisites**
   ```powershell
   Invoke-AtomicTest T1082 -CheckPrereqs
   ```

3. **Get prerequisites if needed**
   ```powershell
   Invoke-AtomicTest T1082 -GetPrereqs
   ```

4. **Run the test**
   ```powershell
   Invoke-AtomicTest T1082
   # or specific test number
   Invoke-AtomicTest T1082 -TestNumbers 1
   ```

5. **Immediately check telemetry**
   - Sysmon Event ID 1 (Process Create)
   - Security Event ID 4688
   - Wazuh dashboard for related alerts
   - Optional: Wireshark if network activity is expected

6. **Cleanup**
   ```powershell
   Invoke-AtomicTest T1082 -Cleanup
   ```

7. **Document**
   - Technique ID and name
   - Test number executed
   - Timestamp
   - What appeared in Sysmon / Event Logs / SIEM
   - Whether an existing detection fired
   - Any gaps observed

---

## 7. Detection Validation Mindset

For every atomic test ask:

1. Did Sysmon capture the process creation with full command line?
2. Did any Wazuh rule fire?
3. Is the activity visible in the Security log?
4. Would a Tier-1 analyst notice this in a real alert queue?
5. What detection rule could I write or improve based on this?

This turns Atomic Red Team from a simple “run attacker commands” exercise into real detection engineering practice.

---

## 8. Suggested Lab Progression

1. **Discovery techniques** (T1082, T1033, T1087, T1016, T1049)
2. **Execution – PowerShell** (T1059.001)
3. **Defence Evasion – basic** (timed stomp, indicator removal on host, etc.)
4. **Credential Access** (only carefully, in isolated snapshots)
5. **Persistence** (simple run keys, scheduled tasks — clean up thoroughly)
6. **Map results** to your existing Wazuh rules and note coverage gaps

---

## 9. Documentation Tips for Portfolio

For each technique you test, record:
- ATT&CK ID and name
- Exact atomic test executed
- Screenshot of the command / output
- Screenshot of the corresponding Sysmon Event ID 1 (or relevant event)
- Whether a SIEM alert fired
- Short note on detection gap or confirmation

A table of 5–8 tested techniques with detection results makes a strong portfolio artefact.

---

## 10. References

- Atomic Red Team GitHub: https://github.com/redcanaryco/atomic-red-team
- Invoke-AtomicRedTeam: https://github.com/redcanaryco/invoke-atomicredteam
- MITRE ATT&CK Navigator (for mapping coverage)
- Your existing Sysmon, Wazuh, and Windows Event Log lab documentation

---

**Document prepared for practical SOC and detection engineering skill development.**  
Always run Atomic Red Team only inside isolated lab environments.
