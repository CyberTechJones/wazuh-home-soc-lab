# Lessons Learned

## Overview

Building this authorized Wazuh home SOC lab provided practical experience with endpoint telemetry, centralized event collection, alert investigation, evidence correlation, and security-event classification.

The project demonstrated that effective security monitoring requires more than installing a SIEM. Each telemetry source must be configured, validated locally, confirmed centrally, and tested through controlled activity.

## Telemetry Collection and Alert Generation

A collected event does not automatically generate an alert.

During the Sysmon Process-Creation scenario, Sysmon recorded the controlled Notepad process locally and the Windows Wazuh agent forwarded the event to SIEM-01. However, the event did not appear in the Wazuh Threat Hunting alert index because the benign Notepad execution did not match an alert-generating rule.

Temporary Wazuh JSON archiving confirmed that the event reached SIEM-01.

This demonstrated the distinction between:

1. Endpoint telemetry generation
2. Agent collection
3. Centralized event receipt
4. Detection-rule evaluation
5. Alert generation

The absence of an alert does not necessarily mean that telemetry collection failed.

## Local and Centralized Correlation

Local endpoint evidence should be compared with centralized SIEM evidence before reaching a conclusion.

Unique command-line markers made this process more reliable. Examples included:

```text
SOC-Lab-Scenario1.txt
SOC-Lab-Scenario4-Retest
```

The markers helped identify the same controlled activity across PowerShell output, local Sysmon records, and centralized Wazuh events.

Useful correlation fields included:

- Event timestamp
- Hostname
- Wazuh agent name
- Event ID
- Process image
- Full command line
- Parent process
- User context
- Integrity level
- File path
- Hash values
- Wazuh rule ID, description, and level

Matching several fields provided stronger evidence than relying on a timestamp or rule description alone.

## Windows Authentication Analysis

Windows Security Event ID `4625` confirms that an authentication attempt failed. The event does not independently prove brute force, password spraying, credential theft, or malicious intent.

The failed-logon investigation demonstrated the importance of reviewing:

- Target account
- Target domain
- Logon Type
- Failure reason
- Status and sub-status codes
- Workstation name
- Source address and port
- Number and timing of attempts
- Related successful logons
- Account-lockout policy

The representative event used Logon Type `2`, which was consistent with an interactive Windows sign-in.

The status values provided additional context:

```text
Status:     0xC000006D
Sub-status: 0xC000006A
```

The three Wazuh alerts matched the three intentional failed-password attempts. This supported an authorized benign true-positive classification rather than a brute-force conclusion.

The test also showed why account-lockout settings should be reviewed before generating controlled authentication failures.

## File Integrity Monitoring

A File Integrity Monitoring alert proves that a monitored file changed. The alert does not automatically establish who changed the file, why the file changed, or whether the change was malicious.

The controlled FIM scenario demonstrated detection of:

- File creation
- Content modification
- File renaming
- File deletion

Wazuh recorded the following rules:

| File action | Rule ID | Rule level | Description |
|---|---:|---:|---|
| Added | `554` | `5` | File added |
| Modified | `550` | `7` | Integrity checksum changed |
| Deleted | `553` | `7` | File deleted |

The rename did not appear as one event labeled `renamed`. Wazuh represented the action as two closely timed events:

```text
Original path deleted
Renamed path added
```

This demonstrated that filesystem activity may need to be reconstructed from several related events.

The scenario also reinforced that user or process attribution should not be claimed unless the FIM event or supporting telemetry provides that information.

## Process and Command-Line Analysis

Sysmon Process Create Event ID `1` provided important process context, including:

- Process image
- Full command line
- Current directory
- Parent process
- Parent command line
- User context
- Integrity level
- Process ID
- Process GUID
- SHA-256 hash
- Local and UTC timestamps

The command line often provided more investigative value than the executable name alone.

For example, `powershell.exe` is a legitimate Windows utility. The full command line revealed that the controlled PowerShell process executed:

```powershell
Get-LocalUser | Select-Object Name,Enabled,Description
```

Without the command line, an analyst could confirm that PowerShell ran but could not accurately describe what PowerShell did.

## Parent-Child Process Relationships

Parent-child process analysis provided useful execution context.

In Scenario 4, an existing PowerShell process launched a child `powershell.exe` process. Wazuh generated rule `92027`, which identified the PowerShell parent-child relationship.

A parent-child relationship can increase or reduce suspicion depending on context. Examples that may require additional review include:

```text
winword.exe → powershell.exe
excel.exe → cmd.exe
browser.exe → script interpreter
```

However, an unusual relationship does not independently prove malicious behavior. The command line, user context, file path, integrity level, authorization, and related activity must also be considered.

## PowerShell Analysis

PowerShell is a legitimate administrative tool and should not automatically be treated as malicious.

The controlled `Get-LocalUser` command was:

- Read-only
- Authorized
- Executed within the lab
- Limited to local-account information
- Not associated with account modification
- Not associated with credential access
- Not associated with persistence

Wazuh correctly generated a process alert, but the underlying activity remained benign.

This demonstrated the importance of separating:

```text
Observed behavior
Detection result
User intent
Incident status
```

A detection may be accurate even when the underlying activity is authorized.

## MITRE ATT&CK Assessment

MITRE ATT&CK mappings describe observable behavior rather than malicious intent.

The project distinguished between:

- **Wazuh-assigned mappings:** Displayed in the matching Wazuh alert
- **Analyst-assessed mappings:** Added after reviewing the observed behavior
- **Unsupported mappings:** Not justified by the available evidence

During the PowerShell Local-Account Discovery scenario, Wazuh assigned:

- **Tactic:** Execution
- **Technique:** Command and Scripting Interpreter: PowerShell
- **Technique ID:** `T1059.001`

The `Get-LocalUser` behavior was also consistent with an analyst assessment of:

- **Tactic:** Discovery
- **Technique:** Account Discovery: Local Account
- **Technique ID:** `T1087.001`

The report clearly distinguished the Wazuh-assigned mapping from the analyst-assessed mapping.

This avoided presenting an analyst interpretation as though it had been generated by Wazuh.

## Event, Alert, and Incident Classification

The project reinforced the difference between an event, an alert, and an incident.

### Event

A structured record describing observed activity.

Example:

```text
Sysmon Event ID 1 recorded a new PowerShell process.
```

### Alert

An event or group of events that matched detection logic.

Example:

```text
Wazuh rule 92027 generated an alert for a PowerShell parent-child process.
```

### Incident

Validated unauthorized activity requiring response or escalation.

None of the four controlled scenarios resulted in a security incident because the activities were authorized, limited, and performed within the lab.

## Alert Disposition

The project used the following classifications:

### Authorized Benign Activity

Authorized and harmless activity was observed, but no security alert was generated.

Scenario 1 used this classification because the Notepad event reached Wazuh but did not match an alert-generating rule.

### Authorized Benign True Positive

The detection correctly identified behavior that was intentionally generated during an authorized test.

Scenarios 2, 3, and 4 used this classification because Wazuh generated alerts that accurately represented the controlled activity.

### False Positive

A false positive would occur if detection logic incorrectly characterized benign activity as suspicious or malicious.

The completed scenarios were not classified as false positives because the alerts accurately identified the underlying behavior.

## Severity and Confidence

Severity should reflect the observed risk and potential impact, not only the Wazuh rule level.

The project used:

- **Informational:** Authorized activity with no meaningful security risk
- **Low:** An alert required analyst review but presented limited risk
- **Medium:** Activity was suspicious or unexplained
- **High:** Evidence strongly supported unauthorized or harmful activity
- **Critical:** Confirmed compromise created significant operational impact

Confidence reflected the quality and consistency of the evidence:

- **Low confidence:** Evidence was incomplete or conflicting
- **Medium confidence:** Several indicators supported the conclusion, but important questions remained
- **High confidence:** Local and centralized evidence were consistent and directly supported the conclusion

The completed scenarios received High confidence because the activity was intentionally generated, locally validated, and correlated with Wazuh telemetry.

## Evidence Limitations

Missing telemetry should be documented as a limitation rather than replaced with assumptions.

Examples included:

- User attribution unavailable
- Process attribution unavailable
- Source address not populated
- Command line incomplete
- Raw event unavailable
- Event archived but not present in the alert index
- Time-zone differences
- Agent-forwarding delay
- Wazuh indexing delay

The investigation reports separated:

- What the evidence proved
- What the evidence suggested
- What the evidence did not establish

This reduced the risk of overstating findings.

## Timestamp Analysis

Endpoint and Wazuh timestamps were not always identical.

Differences could result from:

- Local time versus UTC
- Agent-forwarding delay
- Manager processing time
- Detection-rule evaluation
- Dashboard indexing delay

Timestamps were compared together with the hostname, Event ID, command line, process image, file path, account context, and unique test marker.

A small timestamp difference did not invalidate a correlation when the remaining event fields matched.

## Safe Testing and Recovery

Controlled security testing should include safeguards, validation, and rollback procedures.

Important practices included:

- Taking VirtualBox snapshots before major configuration changes
- Backing up `ossec.conf`
- Validating XML before restarting the Wazuh agent
- Confirming the agent remained Active
- Remaining below account-lockout thresholds
- Using dedicated test directories
- Using harmless files and read-only commands
- Restricting tests to authorized systems
- Disabling temporary JSON archiving after validation
- Preserving known-good configuration backups

These practices made the lab safer and helped separate configuration problems from detection problems.

## Troubleshooting

Troubleshooting was also an important part of the project.

Examples included:

- Confirming TCP connectivity to the Wazuh manager on port `1514`
- Verifying that `wazuh-remoted` was listening
- Checking Windows service status
- Reviewing `ossec.log`
- Confirming that a custom FIM path was loaded
- Distinguishing old log errors from current service health
- Correcting dashboard query syntax
- Repeating controlled tests when SIEM-01 was offline
- Verifying that screenshots represented the same process execution

The troubleshooting process demonstrated that a valid XML file does not automatically prove that a configuration was loaded successfully. Service logs and fresh controlled events were required for complete validation.

## Evidence Handling

Public evidence required careful review.

The project excluded or redacted:

- Passwords
- Tokens
- Enrollment keys
- Private keys
- Unnecessary IP addresses
- Personal identifiers
- Unrelated event content

Generic lab-only usernames and hostnames were retained when they provided useful investigative context.

Unredacted evidence should remain outside the public repository.

## Documentation and Reproducibility

Screenshots alone were not sufficient documentation.

Each completed scenario included:

- Objective
- Environment
- Safety controls
- Controlled test activity
- Local evidence
- Centralized evidence
- Investigation and correlation
- Findings
- Evidence limitations
- MITRE ATT&CK assessment
- Severity and confidence
- Final disposition
- Cleanup
- Lessons learned

The playbooks converted those completed investigations into reusable procedures for future authentication, process, PowerShell, Sysmon, and File Integrity Monitoring alerts.

## Final Takeaway

The most important lesson was that security telemetry must be interpreted in context.

An event proves that activity was recorded.

An alert proves that detection logic matched observed behavior.

Neither an event nor an alert automatically proves malicious intent or a security incident.

Reliable SOC conclusions require:

- Validation of the underlying event
- Correlation of endpoint and centralized evidence
- Timeline development
- Scope determination
- Legitimate-activity analysis
- Documentation of evidence limitations
- Evidence-based severity and confidence
- A defensible final disposition
