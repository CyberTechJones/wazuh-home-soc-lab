# Wazuh Home SOC Lab

An isolated three-VM Security Operations Center lab for endpoint monitoring, controlled detection testing, Wazuh alert investigation, and MITRE ATT&CK analysis.

## Project Overview

I built this home SOC lab to gain hands-on experience with:

- Wazuh administration
- Windows and Linux endpoint monitoring
- Sysmon telemetry
- Windows authentication analysis
- File Integrity Monitoring
- PowerShell investigation
- Endpoint-to-SIEM correlation
- MITRE ATT&CK assessment
- Alert triage and final disposition

The project includes four completed detection scenarios and four reusable SOC investigation playbooks.

All testing was authorized, non-destructive, and restricted to virtual machines that I own and operate.

## Architecture

| Asset | Operating System | Role |
|---|---|---|
| **SIEM-01** | Ubuntu Linux | Wazuh manager, indexer, and dashboard |
| **ENDPOINT-01** | Windows 11 | Monitored endpoint with Sysmon |
| **TEST-01** | Kali Linux | Monitored Linux system and controlled testing platform |

[View the lab architecture and network design](./architecture/README.md)

## Technologies

- Wazuh
- Sysmon
- Windows 11
- Kali Linux
- Ubuntu Linux
- Windows PowerShell
- Oracle VirtualBox
- Windows Security Event Log
- Wazuh File Integrity Monitoring
- MITRE ATT&CK
- GitHub Markdown

## Completed Detection Scenarios

| Scenario | Key Evidence | Final Disposition |
|---|---|---|
| [1. Sysmon Process-Creation Validation](./detection-scenarios/01-sysmon-process-creation/README.md) | Sysmon Event ID `1` and centralized Wazuh archive record | Authorized benign activity |
| [2. Controlled Failed Windows Logons](./detection-scenarios/02-failed-windows-logons/README.md) | Windows Event ID `4625` and Wazuh rule `60122` | Authorized benign true positive |
| [3. File Integrity Monitoring](./detection-scenarios/03-file-integrity-monitoring/README.md) | Wazuh FIM rules `550`, `553`, and `554` | Authorized benign true positive |
| [4. PowerShell Local-Account Discovery](./detection-scenarios/04-powershell-account-discovery/README.md) | Sysmon Event ID `1` and Wazuh rule `92027` | Authorized benign true positive |

[View the detection scenario roadmap](./detection-scenarios/README.md)

## Featured Investigation

### PowerShell Local-Account Discovery

I executed the read-only `Get-LocalUser` cmdlet through a child PowerShell process on ENDPOINT-01.

The controlled command included the unique marker:

```text
SOC-Lab-Scenario4-Retest
```

Sysmon recorded the activity as Process Create Event ID `1`, including:

- Full PowerShell command line
- Parent-child process relationship
- User context
- High integrity level
- SHA-256 process hash
- Local and UTC timestamps

Wazuh received the matching event and generated a level `4` alert under rule `92027`.

Wazuh mapped the event to:

```text
T1059.001: Command and Scripting Interpreter: PowerShell
```

The behavior of `Get-LocalUser` was also assessed as consistent with:

```text
T1087.001: Account Discovery: Local Account
```

The activity was classified as an **authorized benign true positive** with **Low severity** and **High confidence**.

[View the complete PowerShell investigation](./detection-scenarios/04-powershell-account-discovery/README.md)

## Key Results

### Process Telemetry

Sysmon Event ID `1` captured process images, full command lines, parent processes, integrity levels, user context, and SHA-256 hashes.

### Authentication Monitoring

Windows recorded three controlled failed interactive sign-ins as Event ID `4625`. Wazuh generated three corresponding level `5` alerts under rule `60122`.

### File Integrity Monitoring

Wazuh detected the complete controlled file lifecycle:

```text
Created → Modified → Renamed → Deleted
```

The rename appeared as deletion of the original path followed by addition of the renamed path.

### Event Versus Alert Visibility

A benign Notepad event reached Wazuh but did not generate a visible Threat Hunting alert. Temporary JSON archiving confirmed centralized collection and demonstrated that event receipt and alert generation are separate stages.

## Investigation Playbooks

| Playbook | Primary Use |
|---|---|
| [General Alert Triage](./playbooks/alert-triage-playbook.md) | Alert validation, classification, escalation, and closure |
| [Failed Windows Logons](./playbooks/failed-logon-playbook.md) | Windows Event ID `4625` and authentication-failure investigation |
| [File Integrity Monitoring](./playbooks/file-integrity-playbook.md) | File creation, modification, rename, and deletion investigation |
| [Suspicious Process Activity](./playbooks/suspicious-process-playbook.md) | Sysmon Event ID `1`, PowerShell, command-line, and parent-process analysis |

[View the SOC investigation playbook index](./playbooks/README.md)

## SOC Skills Demonstrated

- Wazuh manager and agent administration
- Windows and Linux endpoint monitoring
- Sysmon deployment and analysis
- Windows Security event analysis
- Process and command-line investigation
- Parent-child process analysis
- Failed-authentication investigation
- File Integrity Monitoring
- PowerShell activity analysis
- File-hash review
- Timeline development
- Endpoint-to-SIEM correlation
- Wazuh rule validation
- MITRE ATT&CK assessment
- Severity and confidence Document what the evidence proves and does not prove.
9. Review MITRE ATT&CK mappings.
10. Assign severity, confidence, and final disposition.
11. Complete cleanup and preserve sanitized evidence.

## Key Lessons Learned

- Event collection and alert generation are separate stages.
- An alert can be accurate while the underlying activity remains authorized and benign.
- Local endpoint evidence should be correlated with centralized Wazuh records.
- Unique markers improve cross-system correlation.
- Windows Event ID `4625` requires Logon Type and status-code interpretation.
- File renames may appear as related deletion and addition events.
- PowerShell usage alone does not establish malicious intent.
- Wazuh-assigned and analyst-assessed MITRE ATT&CK mappings should be distinguished.
- Severity should reflect evidence and risk, not only the Wazuh rule level.
- Missing telemetry should be documented as a visibility limitation.

./documentation/lessons-learned.md

## Repository Guide

```text
wazuh-home-soc-lab/
├── README.md
├── architecture/
├── setup/
├── detection-scenarios/
├── playbooks/
└── documentation/
```

| Directory | Contents |
|---|---|
| ./architecture/ | Network design and lab architecture |
| ./setup/ | Asset inventory and setup evidence |
| ./detection-scenarios/ | Four evidence-backed investigations |
| ./playbooks/ | Four reusable SOC investigation procedures |
| ./documentation/ | Project-wide lessons learned |

## Recommended Review Path

For a concise review of the project:

1. ./architecture/README.md
2. ./detection-scenarios/README.md
3. ./detection-scenarios/04-powershell-account-discovery/README.md
4. ./playbooks/alert-triage-playbook.md
5. ./documentation/lessons-learned.md

## Project Limitations

This is a controlled home lab rather than a production SOC environment.

Current limitations include:

- One Windows endpoint
- One Linux testing system
- No production users or business data
- Limited event-retention capacity
- Primarily endpoint-focused telemetry
- Intentionally generated test activity
- No network IDS sensor
- No continuous 24-hour monitoring requirement

These limitations are documented to avoid overstating the project’s operational scope.

## Future Improvements

- Enable PowerShell Script Block Logging with Event ID `4104`.
- Create and test a custom Wazuh rule for repeated failed logons.
- Collect Microsoft Defender telemetry.
- Test authorized remote authentication from TEST-01.
- Measure endpoint-to-Wazuh detection delay.
- Evaluate selective event archiving and retention.
- Add Suricata network telemetry.

## Authorization and Ethics

All activity documented in this repository was performed in an isolated Oracle VirtualBox environment on systems that I own and operate.

The project did not target external systems, production infrastructure, unauthorized accounts, or third-party data.

Passwords, private keys, tokens, enrollment credentials, and other secrets are excluded. Screenshots were reviewed and sanitized before publication.

## Project Status

- [x] Three-VM architecture documented
- [x] Wazuh manager deployed
- [x] Windows and Kali agents enrolled
- [x] Sysmon deployed on ENDPOINT-01
- [x] Four detection scenarios completed
- [x] Four investigation playbooks completed
- [x] Lessons learned documented
- [x] Public evidence reviewed and sanitized

**Status:** Complete
