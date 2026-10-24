# Detection Scenario Roadmap

## Overview

This directory documents controlled security tests performed within my authorized home SOC lab. Each scenario validates a different security-monitoring capability and applies a repeatable workflow for telemetry review, alert investigation, evidence correlation, and final disposition.

All testing is non-destructive and restricted to virtual machines that I own and operate within the controlled Oracle VirtualBox lab environment.

## Lab Systems

- **TEST-01:** Kali Linux testing and monitored Linux system
- **ENDPOINT-01:** Monitored Windows 11 endpoint
- **SIEM-01:** Wazuh Security Information and Event Management platform

TEST-01 generates authorized test activity when required. Wazuh agents on TEST-01 and ENDPOINT-01 forward configured telemetry to SIEM-01 for centralized analysis.

## Investigation Methodology

Each completed scenario documents:

1. Objective and authorized scope
2. Safety controls and prerequisites
3. Controlled test activity
4. Expected and observed data sources
5. Local endpoint evidence
6. Centralized Wazuh evidence
7. Investigation and correlation
8. Evidence-supported findings
9. MITRE ATT&CK assessment
10. Severity, confidence, and final disposition
11. Response considerations
12. Cleanup and lessons learned

## Scenario Status

| Scenario | Status | Primary SOC Skill |
|---|---|---|
| [Scenario 1: Sysmon Process-Creation Validation](./01-sysmon-process-creation/README.md) | Complete | Process and telemetry analysis |
| [Scenario 2: Controlled Failed Windows Logons](./02-failed-windows-logons/README.md) | Complete | Authentication investigation |
| [Scenario 3: File Integrity Monitoring](./03-file-integrity-monitoring/README.md) | Complete | File-change investigation |
| Scenario 4: PowerShell Local-Account Discovery | Planned | Command-line and discovery analysis |

---

## Scenario 1: Sysmon Process-Creation Validation

### Objective

Validate the complete telemetry path for a controlled process launch on ENDPOINT-01.

### Controlled Activity

Launched Windows Notepad using the unique command-line marker `C:\SOC-Lab-Scenario1.txt`.

### Data Sources

- `Microsoft-Windows-Sysmon/Operational`
- Sysmon Event ID `1`
- Windows Wazuh agent telemetry
- Wazuh JSON event archive

### Key Result

Sysmon recorded the process locally, and the Windows Wazuh agent forwarded the corresponding event to SIEM-01. The exact event was correlated in the Wazuh JSON archive using the unique command-line marker.

The benign Notepad execution did not generate a visible Threat Hunting alert, demonstrating that successful event collection and alert generation are separate stages.

### Skills Demonstrated

- Sysmon Event ID `1` analysis
- Windows process and command-line analysis
- Parent-process review
- File-hash review
- Endpoint-to-SIEM telemetry validation
- Timeline correlation
- Event-versus-alert classification

### Final Disposition

- **Status:** Complete
- **Classification:** Authorized benign activity
- **Severity:** Informational
- **Confidence:** High
- **Security incident:** No

[View the completed Scenario 1 report](./01-sysmon-process-creation/README.md)

---

## Scenario 2: Controlled Failed Windows Logons

### Objective

Validate the collection, detection, and investigation of controlled Windows authentication failures on ENDPOINT-01.

### Controlled Activity

Entered an incorrect password three times for a dedicated local lab account at the Windows lock screen.

### Data Sources

- Windows Security event log
- Windows Security Event ID `4625`
- Windows Wazuh agent telemetry
- Wazuh Threat Hunting alerts

### Key Result

Windows recorded three failed interactive sign-ins as Event ID `4625`. Wazuh generated three corresponding level `5` alerts under rule `60122`.

The representative local event used Logon Type `2` and recorded the failure reason as an unknown username or incorrect password. The alert count matched the controlled test count.

### Skills Demonstrated

- Windows authentication-event analysis
- Failed-logon investigation
- Logon-type interpretation
- Status and sub-status code analysis
- Wazuh rule validation
- Endpoint-to-SIEM correlation
- Severity and confidence assessment
- Benign true-positive classification

### Final Disposition

- **Status:** Complete
- **Classification:** Authorized benign true positive
- **Severity:** Low
- **Confidence:** High
- **Security incident:** No

[View the completed Scenario 2 report](./02-failed-windows-logons/README.md)

---

## Scenario 3: File Integrity Monitoring

### Objective

Validate real-time detection and investigation of controlled file creation, modification, renaming, and deletion on ENDPOINT-01.

### Controlled Activity

Configured Wazuh File Integrity Monitoring for:

`C:\SOC-Lab\FIM-Test`

### Key Result

Wazuh detected the complete controlled file lifecycle:

`Added → Modified → Renamed → Deleted`

Rule `554` detected file additions, rule `550` detected the content modification, and rule `553` detected deletions. Wazuh represented the rename as deletion of the original path followed by addition of the renamed path.

### Final Disposition

- **Status:** Complete
- **Classification:** Authorized benign true positive
- **Severity:** Low
- **Confidence:** High
- **Security incident:** No

[View the completed Scenario 3 report](./03-file-integrity-monitoring/README.md)

---
