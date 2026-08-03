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

This report documents the practical use of **Atomic Red Team** to emulate selected MITRE ATT&CK techniques in a controlled Windows lab environment. The tests were executed on a Windows endpoint instrumented with Sysmon, with telemetry also available via Windows Event Logs and the Wazuh SIEM.

The primary objective was not simply to run attacker commands, but to observe how each technique appears in endpoint and SIEM telemetry, and to evaluate detection visibility. This approach builds the detection engineering mindset required for modern SOC work.

**Key outcomes achieved:**
- Successful installation and execution of Atomic Red Team tests
- Execution of multiple discovery and execution techniques
- Observation of corresponding Sysmon (Event ID 1) and Security log events
- Mapping of activity to MITRE ATT&CK techniques
- Initial assessment of detection coverage and gaps
- Structured documentation suitable for portfolio use

---

## 2. Lab Objectives

| # | Objective | Status |
|---|-----------|--------|
| 1 | Install Invoke-AtomicRedTeam and atomics | Achieved |
| 2 | Execute selected low-impact atomic tests safely | Achieved |
| 3 | Capture process creation telemetry with Sysmon | Achieved |
| 4 | Review related Windows Security events | Achieved |
| 5 | Map tests to MITRE ATT&CK techniques | Achieved |
| 6 | Assess whether existing detections would fire | Achieved |
| 7 | Document results and gaps | Achieved |

---

## 3. Lab Environment

- Windows 10/11 lab machine (domain-joined preferred)
- Sysmon installed with a quality configuration
- Windows Security Event Log auditing enabled
- Optional but recommended: Wazuh agent forwarding Sysmon and Security logs
- Atomic Red Team installed via Invoke-AtomicRedTeam framework

This environment allows immediate correlation between the technique executed and the resulting telemetry.

---

## 4. Implementation Summary

### 4.1 Installation

Invoke-AtomicRedTeam was installed and the atomics library downloaded:

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

Installation was verified by listing technique details and checking prerequisites.

### 4.2 Test Execution Approach

Tests were selected starting with low-impact discovery techniques. For each test the following workflow was followed:

1. Review technique details (`-ShowDetails`)
2. Check and satisfy prerequisites
3. Execute the test
4. Immediately inspect Sysmon Event ID 1 and relevant Security events
5. Check Wazuh for any related alerts
6. Run cleanup where available
7. Record results

### 4.3 Techniques Practised (Examples)

| ATT&CK ID | Technique | Focus of Test | Primary Telemetry Observed |
|-----------|-----------|---------------|---------------------------|
| T1082 | System Information Discovery | systeminfo / hostname style commands | Sysmon Event ID 1 |
| T1033 | System Owner/User Discovery | whoami and related commands | Sysmon Event ID 1, Security 4688 |
| T1087 | Account Discovery | net user / net localgroup | Sysmon Event ID 1 |
| T1016 | System Network Configuration Discovery | ipconfig / route | Sysmon Event ID 1 |
| T1059.001 | Command and Scripting Interpreter: PowerShell | Simple PowerShell execution | Sysmon 1, PowerShell logging |

These techniques produce clear, easily observable process-creation events and form a safe starting point for detection validation.

---

## 5. Results and Evidence

### 5.1 Telemetry Visibility

| Source | Visibility into Atomic Tests |
|--------|------------------------------|
| Sysmon Event ID 1 | Excellent – full command line, parent process, user, hashes |
| Windows Security 4688 | Present but limited command-line detail compared with Sysmon |
| Wazuh SIEM | Events available when agent is configured; rule coverage depends on existing detections |
| Network (Wireshark) | Limited for pure discovery tests; more relevant for later techniques |

### 5.2 Detection Engineering Observations

- Discovery techniques generate high-fidelity process creation events that are straightforward to detect with command-line aware rules.
- Parent-child process relationships visible in Sysmon provide strong context for tuning.
- Running the same test multiple times allows verification that detections are reliable and not one-off.
- Gaps become obvious when a technique produces clear telemetry but no SIEM alert fires — this is valuable input for rule writing.

---

## 6. Investigation / Validation Case Study

**Scenario:** Execute a System Owner/User Discovery technique (T1033) and validate telemetry + detection visibility.

**Steps performed:**

1. Confirmed Sysmon was running and Wazuh agent was healthy.
2. Executed an atomic test for T1033 (whoami-style discovery).
3. Immediately filtered Sysmon Operational log for Event ID 1 containing the relevant Image/CommandLine.
4. Located the corresponding Security Event ID 4688.
5. Checked Wazuh dashboard for any alert related to the activity.
6. Recorded whether a detection fired and noted the quality of the available telemetry.
7. Cleaned up and documented the result.

**Outcome:** The technique produced clear, high-quality Sysmon process creation events with full command line. This confirmed that the telemetry foundation is solid and highlighted where detection rules could be added or improved.

**Key Learning:** Atomic Red Team is most valuable when treated as a detection validation tool rather than just an attack simulation library. The cycle “run technique → inspect telemetry → assess detection → document gap or success” builds real detection engineering skill.

---

## 7. Challenges Encountered and Resolutions

| Challenge | Resolution |
|-----------|------------|
| Ensuring tests stay low-impact | Started exclusively with discovery techniques and used VM snapshots |
| Correlating exact test to events | Noted precise timestamps and used unique command-line strings where possible |
| SIEM rule coverage initially low | Expected for a new lab; treated gaps as future rule-writing opportunities |
| Cleanup after tests | Used `-Cleanup` parameter and manual verification |

---

## 8. Lessons Learned

1. **Telemetry first, detection second** — High-quality Sysmon data makes subsequent detection work far easier.
2. **Start simple** — Discovery techniques teach the workflow without the risk of more aggressive tests.
3. **Repeatability matters** — Being able to re-run the same test and see consistent telemetry builds confidence.
4. **Gaps are useful** — A technique that is fully visible in logs but generates no alert is a clear and actionable finding.
5. **Documentation closes the loop** — Recording technique, telemetry, and detection result turns testing into portfolio evidence and improves future tuning.

---

## 9. Recommendations and Next Steps

- Expand testing to additional execution and defence-evasion techniques.
- Write or refine Wazuh rules specifically for the techniques already tested.
- Use ATT&CK Navigator to visualise current coverage.
- Combine with the Wireshark lab for techniques that generate network traffic.
- Create a simple detection-coverage tracking table (Technique → Telemetry seen → Alert fired → Notes).
- Progress carefully into credential access and persistence tests only inside snapshots.

---

## 10. Conclusion

The Atomic Red Team lab successfully introduced controlled ATT&CK technique emulation and detection validation into the existing blue-team lab environment. By executing selected tests and immediately examining Sysmon, Event Log, and SIEM telemetry, practical understanding of how attacker techniques appear in real data was developed.

This work directly supports the skills required for detection engineering and proactive SOC work. Together with the Wazuh SIEM, Sysmon, Windows Event Log, Active Directory, and Wireshark labs, it completes a strong foundation for Junior SOC Analyst readiness.

---

## Appendix – Useful Commands

```powershell
# Show technique details
Invoke-AtomicTest T1082 -ShowDetails

# Check prerequisites
Invoke-AtomicTest T1082 -CheckPrereqs

# Run test
Invoke-AtomicTest T1082

# Run specific test number
Invoke-AtomicTest T1082 -TestNumbers 1

# Cleanup
Invoke-AtomicTest T1082 -Cleanup
```

---

**End of Report**

*This document is intended for professional portfolio and interview use. Screenshots of atomic test execution, corresponding Sysmon events, and SIEM views can be added as testing progresses.*
