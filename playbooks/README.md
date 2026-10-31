# SOC Investigation Playbooks

## Overview

This directory contains repeatable investigation procedures developed from completed detection scenarios in my authorized Wazuh home SOC lab.

The detection scenarios document specific controlled tests and their results. These playbooks convert those investigations into reusable workflows for validating alerts, determining scope, assessing available evidence, assigning severity and confidence, and documenting final dispositions.

## Available Playbooks

| Playbook | Status | Related Scenarios |
|---|---|---|
| ./alert-triage-playbook.md | Complete | All scenarios |
| ./failed-logon-playbook.md | Complete | Scenario 2 |
| ./file-integrity-playbook.md | Complete | Scenario 3 |
| ./suspicious-process-playbook.md | Complete | Scenarios 1 and 4 |

## Investigation Methodology

The playbooks follow a consistent investigation process:

1. Review the alert metadata.
2. Validate the underlying endpoint event.
3. Identify the affected asset, user, process, or file.
4. Build the event timeline.
5. Determine the investigation scope.
6. Review related endpoint and Wazuh telemetry.
7. Evaluate legitimate explanations.
8. Document evidence limitations.
9. Review applicable MITRE ATT&CK mappings.
10. Assign severity and confidence.
11. Determine whether escalation or response is required.
12. Document the final disposition and closure decision.

## General Alert Triage

The [General Alert Triage Playbook](./alert-triage-playbook.md) provides the shared workflow used to review, validate, classify, escalate, and close Wazuh alerts.

The playbook includes:

- Alert-metadata review
- Underlying-event validation
- Asset and user identification
- Timeline development
- Scope determination
- Legitimate-activity analysis
- Evidence-limitation documentation
- MITRE ATT&CK review
- Severity and confidence assessment
- Escalation and closure criteria
- Investigation record template

This playbook applies across authentication, process, PowerShell, Sysmon, and File Integrity Monitoring investigations.

## Failed Windows Logons

The [Failed Windows Logon Investigation Playbook](./failed-logon-playbook.md) provides a repeatable procedure for investigating Windows Security Event ID `4625`.

The playbook covers:

- Target-account context
- Logon Type interpretation
- Status and sub-status codes
- Authentication package
- Source and workstation information
- Event frequency
- Related successful logons
- Account-lockout context
- Brute-force and password-spraying indicators
- Benign explanations and escalation criteria

The procedure was validated through the completed controlled failed-logon investigation in Scenario 2.

## File Integrity Monitoring

The [File Integrity Monitoring Investigation Playbook](./file-integrity-playbook.md) provides a structured procedure for investigating file creation, modification, rename, and deletion alerts.

The playbook covers:

- Wazuh `syscheck` events
- File paths and actions
- Previous and current hashes
- File-size and attribute changes
- Text-content differences, when available
- User and process attribution
- Rename correlation
- File-change timelines
- Authorized-change validation
- Response and escalation criteria

The procedure was validated through the completed File Integrity Monitoring investigation in Scenario 3.

## Suspicious Process Activity

The [Suspicious Process Investigation Playbook](./suspicious-process-playbook.md) provides a workflow for investigating Sysmon Process Create Event ID `1` and related Wazuh alerts.

The playbook covers:

- Process image and executable path
- Full command line
- Parent-child process relationships
- User and integrity context
- SHA-256 process hashes
- PowerShell and script-interpreter activity
- Encoded or obfuscated commands
- Discovery behavior
- MITRE ATT&CK assessment
- Related process, file, authentication, and network telemetry

The procedure was validated through the Sysmon Process-Creation and PowerShell Local-Account Discovery investigations in Scenarios 1 and 4.

## Alert Classification

The following terminology is used throughout the playbooks:

- **Event:** A structured record describing observed system activity.
- **Alert:** An event or group of events that matched detection logic.
- **True positive:** Detection logic correctly identified the observed behavior.
- **Authorized benign activity:** Authorized and non-malicious activity that was observed but did not generate an alert.
- **Authorized benign true positive:** Detection logic correctly identified behavior intentionally generated during an authorized test.
- **False positive:** Detection logic incorrectly classified benign activity as suspicious or malicious.
- **Suspicious activity:** Activity requiring further investigation because the available evidence is inconclusive.
- **Confirmed incidentwork

### Informational

Expected or authorized activity with no meaningful security risk.

Examples include benign telemetry validation and expected system activity that did not generate a security alert.

### Low

Activity that requires analyst review but presents limited risk.

Examples include:

- A small number of failed logons
- An authorized File Integrity Monitoring change
- Controlled PowerShell execution
- A correctly generated alert for authorized test activity

### Medium

Suspicious or unexplained activity requiring further investigation.

Examples include:

- Repeated authentication failures from an unexpected source
- Unapproved PowerShell discovery activity
- Unexpected changes to monitored application files
- An unusual parent-child process relationship

### High

Evidence strongly supports unauthorized or harmful activity.

Examples include:

- Confirmed credential compromise
- Malicious file modification
- Unauthorized persistence
- Suspicious execution with supporting compromise indicators

### Critical

Confirmed compromise creates significant operational or security impact.

Examples include:

- Ransomware activity
- Destructive activity across multiple systems
- Widespread credential compromise
- Active data exfiltration

Severity should reflect the observed risk and potential impact rather than the Wazuh rule level alone.

## Confidence Framework

- **Low confidence:** Evidence is limited, incomplete, or conflicting.
- **Medium confidence:** Multiple indicators support the conclusion, but important questions remain.
- **High confidence:** Endpoint and centralized evidence are consistent, the timeline is established, and the conclusion is directly supported.

## MITRE ATT&CK Assessment

The playbooks distinguish between:

- **Wazuh-assigned mapping:** A tactic or technique displayed in the matching Wazuh alert.
- **Analyst-assessed mapping:** A tactic or technique assigned after reviewing the observed behavior.
- **Unsupported mapping:** A mapping that is not justified by the available evidence.

MITRE ATT&CK describes observable behavior. A technique mapping does not independently establish malicious intent.

## Evidence Standards

Investigations may use:

- Wazuh rule IDs, descriptions, and levels
- Windows Security events
- Sysmon events
- PowerShell events
- Process images and command lines
- Parent-child process relationships
- User and integrity context
- Authentication details
- File paths and hashes
- File Integrity Monitoring events
- Source and destination information
- Event timelines
- MITRE ATT&CK mappings
- Final alert dispositions

Analysts should clearly distinguish between:

- What the evidence proves
- What the evidence suggests
- What the evidence does not establish

Missing telemetry should be documented as a limitation rather than used to support an unsupported conclusion.

## Evidence Handling

Before evidence is published:

- Passwords, credentials, tokens, keys, and enrollment information are excluded.
- IP and MAC addresses are redacted when unnecessary.
- Personal identifiers are removed when they do not add investigative value.
- Screenshots contain only evidence relevant to the conclusion.
- Screenshot timestamps and fields are verified against the documented event.
- Unredacted evidence remains outside the public repository.
- Claims remain limited to what the available evidence supports.

## Playbook Validation

The playbooks were developed from and validated against four completed investigations:

| Detection Scenario | Primary Investigation Focus | Final Disposition |
|---|---|---|
| ./../detection-scenarios/01-sysmon-process-creation/README.md | Process creation and endpoint-to-SIEM telemetry validation | Authorized benign activity |
| ./../detection-scenarios/02-failed-windows-logons/README.md | Windows authentication failures and Event ID `4625` | Authorized benign true positive |
| ./../detection-scenarios/03-file-integrity-monitoring/README.md | File creation, modification, rename, and deletion | Authorized benign true positive |
| ./../detection-scenarios/04-powershell-account-discovery/README.md | PowerShell execution and local-account discovery | Authorized benign true positive |

## Completion Status

- [x] General alert-triage playbook completed
- [x] Failed-logon playbook completed
- [x] File-integrity playbook completed
- [x] Suspicious-process playbook completed

**Status:** Complete
