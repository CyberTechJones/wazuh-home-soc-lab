# Suspicious Process Investigation Playbook

## Purpose

Provide a repeatable workflow for investigating suspicious or unexpected process-creation alerts using Sysmon Process Create Event ID `1` and Wazuh telemetry.

This playbook helps determine whether a process resulted from authorized administration, normal application behavior, controlled security testing, suspicious execution, or confirmed malicious activity.

## Related Playbooks

Use the general triage workflow for shared investigation steps:

[General Alert Triage Playbook](./alert-triage-playbook.md)

## Validation Scenarios

This playbook was validated through:

- [Scenario 1: Sysmon Process-Creation Validation](../detection-scenarios/01-sysmon-process-creation/README.md)
- [Scenario 4: PowerShell Local-Account Discovery](../detection-scenarios/04-powershell-account-discovery/README.md)

The validation scenarios included:

- A controlled Notepad process launch
- Sysmon Process Create Event ID `1`
- Full command-line capture
- Parent-process analysis
- SHA-256 process hashing
- Controlled PowerShell execution
- Read-only local-account discovery
- Local-to-Wazuh event correlation
- Wazuh rule `92027`
- MITRE ATT&CK review

## Trigger Conditions

Use this playbook when:

- Wazuh generates a process-creation alert.
- Sysmon Event ID `1` records an unexpected process.
- A process launches from an unusual location.
- A process uses a suspicious or unexplained command line.
- PowerShell, Command Prompt, Windows Script Host, or another interpreter executes unexpectedly.
- A trusted process launches an unusual child process.
- A process runs with elevated integrity.
- A process hash is unknown or does not match a trusted version.
- Discovery, persistence, credential access, or defense-evasion behavior is observed.
- The user, parent process, or authorization context cannot be validated.

## Initial Alert Information

> **Usage note:** Copy the following fields into a new investigation record and replace each bracketed placeholder with values from the alert being investigated. Do not permanently replace the placeholders with values from the validation scenarios.

Record:

- **Alert timestamp:** `[ENTER VALUE]`
- **Wazuh rule ID:** `[ENTER VALUE]`
- **Rule description:** `[ENTER VALUE]`
- **Rule level:** `[ENTER VALUE]`
- **Agent name:** `[ENTER VALUE]`
- **Agent ID:** `[ENTER VALUE]`
- **Hostname:** `[ENTER VALUE]`
- **Sysmon Event ID:** `[ENTER VALUE]`
- **User:** `[ENTER VALUE]`
- **Process image:** `[ENTER VALUE]`
- **Command line:** `[ENTER VALUE]`
- **Current directory:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Parent process:** `[ENTER VALUE]`
- **Parent command line:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Integrity level:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Process ID:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Process GUID:** `[ENTER VALUE OR NOT AVAILABLE]`
- **SHA-256 hash:** `[ENTER VALUE OR NOT AVAILABLE]`
- **MITRE ATT&CK mapping:** `[ENTER VALUE OR NOT ASSIGNED]`

## Investigation Workflow

### 1. Validate the Process Event

Confirm that the underlying endpoint event is Sysmon Process Create Event ID `1`.

Review:

- Event timestamp
- Sysmon provider
- Process image
- Full command line
- Current directory
- User context
- Integrity level
- Process ID
- Process GUID
- SHA-256 hash
- Parent image
- Parent command line
- Parent process ID
- Parent process GUID

Do not rely only on the Wazuh rule description. Compare the Wazuh alert with the original Sysmon event.

Record:

- **Sysmon event located:** `[YES/NO]`
- **Event ID confirmed:** `[YES/NO]`
- **Process image confirmed:** `[YES/NO]`
- **Command line complete:** `[YES/NO/PARTIALLY]`
- **Alert accurately represents event:** `[YES/NO/PARTIALLY]`
- **Validation notes:** `[ENTER VALUE]`

### 2. Review the Process Image

Determine:

- Whether the executable name is expected
- Whether the executable path is expected
- Whether the process is digitally signed
- Whether the executable resides in a user-writable directory
- Whether the filename imitates a legitimate Windows process
- Whether the process is part of an approved application
- Whether the executable hash matches a trusted version

Common expected locations include:

```text
C:\Windows\System32
C:\Windows\SysWOW64
C:\Program Files
C:\Program Files (x86)
```

Potentially higher-risk locations include:

```text
C:\Users\<user>\Downloads
C:\Users\<user>\AppData\Local\Temp
C:\Windows\Temp
C:\ProgramData
Publicly writable directories
Removable media
Network shares
```

A process launching from a user-writable directory is not automatically malicious, but it requires additional context.

Record:

- **Expected process name:** `[YES/NO/UNKNOWN]`
- **Expected path:** `[YES/NO/UNKNOWN]`
- **Digitally signed:** `[YES/NO/NOT CHECKED]`
- **User-writable path:** `[YES/NO]`
- **Hash validated:** `[YES/NO/NOT AVAILABLE]`
- **Image assessment:** `[ENTER VALUE]`

### 3. Analyze the Command Line

Review the full command line for:

- Encoded content
- Obfuscated commands
- Download commands
- Remote URLs
- Base64 strings
- Hidden-window options
- Execution-policy bypasses
- Unusual switches
- Credential-related commands
- Account or system discovery commands
- File deletion or modification
- Persistence-related commands
- Unexpected script paths
- Commands launched from temporary directories

Examples requiring additional review include:

```text
-EncodedCommand
-ExecutionPolicy Bypass
-WindowStyle Hidden
Invoke-WebRequest
Invoke-Expression
DownloadString
FromBase64String
Get-Credential
Get-LocalUser
net user
whoami
```

The presence of one command does not independently prove malicious intent. Evaluate authorization, user context, parent process, and related activity.

Record:

- **Command line expected:** `[YES/NO/UNKNOWN]`
- **Suspicious switches present:** `[YES/NO]`
- **Encoded or obfuscated content:** `[YES/NO]`
- **Discovery behavior present:** `[YES/NO]`
- **Network or download behavior present:** `[YES/NO]`
- **Command-line assessment:** `[ENTER VALUE]`

### 4. Analyze the Parent-Child Relationship

Determine whether the parent process would normally launch the observed child.

Expected relationships may include:

```text
explorer.exe → notepad.exe
powershell.exe → powershell.exe during authorized scripting
services.exe → approved Windows service
taskeng.exe → approved scheduled task
```

Relationships requiring additional review may include:

```text
winword.exe → powershell.exe
excel.exe → cmd.exe
outlook.exe → powershell.exe
browser.exe → script interpreter
pdf-reader.exe → cmd.exe
service process → executable in a user-writable directory
```

Record:

- **Parent process:** `[ENTER VALUE]`
- **Child process:** `[ENTER VALUE]`
- **Relationship expected:** `[YES/NO/UNKNOWN]`
- **Parent command line reviewed:** `[YES/NO/NOT AVAILABLE]`
- **Relationship assessment:** `[ENTER VALUE]`

An unusual parent-child relationship can increase suspicion, but the relationship must be evaluated with command-line and user context.

### 5. Review User and Integrity Context

Determine:

- Which account executed the process
- Whether the account is local, domain, service, or system-managed
- Whether the account normally performs the activity
- Whether the process ran with Medium, High, or System integrity
- Whether administrative privileges were expected
- Whether the user confirmed the activity

Record:

- **User:** `[ENTER VALUE]`
- **Account type:** `[LOCAL/DOMAIN/SERVICE/SYSTEM/UNKNOWN]`
- **Integrity level:** `[LOW/MEDIUM/HIGH/SYSTEM/UNKNOWN]`
- **Elevated execution expected:** `[YES/NO/UNKNOWN]`
- **User validation completed:** `[YES/NO/NOT APPLICABLE]`
- **Context assessment:** `[ENTER VALUE]`

High integrity confirms elevated execution. High integrity does not independently prove malicious activity.

### 6. Review the Process Hash

Review available hashes, preferably SHA-256.

Determine:

- Whether the hash matches a known-good executable
- Whether the hash differs from the expected version
- Whether the file was recently created or modified
- Whether internal or approved threat intelligence identifies the hash
- Whether the file is digitally signed
- Whether the path and hash combination are expected

Record:

- **Hash algorithm:** `[ENTER VALUE]`
- **Process hash:** `[ENTER VALUE]`
- **Known-good comparison:** `[MATCH/MISMATCH/NOT CHECKED]`
- **Threat-intelligence result:** `[KNOWN GOOD/KNOWN MALICIOUS/UNKNOWN/NOT CHECKED]`
- **Hash assessment:** `[ENTER VALUE]`

A previously unknown hash is not automatically malicious.

### 7. Build the Process Timeline

Document activity before, during, and after the process execution.

| Time | Process or Event | User | Evidence Source | Analyst Note |
|---|---|---|---|---|
| `[TIME]` | `[PROCESS/EVENT]` | `[USER]` | `[SOURCE]` | `[NOTE]` |
| `[TIME]` | `[PROCESS/EVENT]` | `[USER]` | `[SOURCE]` | `[NOTE]` |
| `[TIME]` | `[PROCESS/EVENT]` | `[USER]` | `[SOURCE]` | `[NOTE]` |

Search for:

- Parent-process creation
- Other children launched by the process
- File creation or modification
- Registry changes
- Network connections
- Authentication events
- PowerShell events
- Scheduled-task activity
- File Integrity Monitoring alerts
- Process termination
- Similar execution on other endpoints

### 8. Determine the Scope

Classify the activity as:

- **Isolated:** One process execution on one endpoint
- **Repeated:** Similar process activity repeated on one endpoint
- **Distributed:** Similar activity across multiple endpoints or users
- **Unknown:** Available evidence is insufficient

Record:

- **Processes affected:** `[ENTER VALUE]`
- **Users affected:** `[ENTER VALUE]`
- **Endpoints affected:** `[ENTER VALUE]`
- **Related alerts:** `[ENTER VALUE]`
- **Scope:** `[ISOLATED/REPEATED/DISTRIBUTED/UNKNOWN]`

### 9. Evaluate Benign Explanations

Potential legitimate explanations include:

- Authorized security testing
- Administrator troubleshooting
- Approved automation
- Software installation or update
- User-launched application
- Scheduled task
- Endpoint-management activity
- Approved PowerShell script
- System-maintenance activity

Validate the explanation using:

- User confirmation
- Change-management records
- Script contents
- Process path
- Parent process
- Command line
- File hash
- Event timeline

Record:

- **Potential legitimate explanation:** `[ENTER VALUE]`
- **Authorization confirmed:** `[YES/NO/UNKNOWN]`
- **Validation method:** `[ENTER VALUE]`
- **Explanation supported:** `[YES/NO/PARTIALLY]`

### 10. Evaluate Suspicious Indicators

Suspicious indicators include:

- Encoded or obfuscated commands
- Execution-policy bypass
- Hidden PowerShell execution
- Download or remote-content retrieval
- Unexpected child processes
- Office applications launching script interpreters
- Execution from temporary or user-writable paths
- Hash mismatch against a trusted version
- Discovery activity without authorization
- Credential-access commands
- Persistence-related changes
- File or registry modifications
- Network activity to unexpected destinations
- Similar activity across multiple endpoints

Evaluate the complete pattern rather than relying on one indicator.

## MITRE ATT&CK Assessment

Review any mapping assigned by Wazuh and compare it with the observed behavior.

Possible behavior categories may include:

- Command and Scripting Interpreter: PowerShell
- Account Discovery
- Process Discovery
- System Information Discovery
- User and Group Discovery
- Ingress Tool Transfer
- Scheduled Task or Job
- Registry Run Keys or Startup Folder

Record:

- **Wazuh-assigned tactic:** `[ENTER VALUE OR NOT ASSIGNED]`
- **Wazuh-assigned technique:** `[ENTER VALUE OR NOT ASSIGNED]`
- **Technique ID:** `[ENTER VALUE OR NOT ASSIGNED]`
- **Analyst-assessed mapping:** `[ENTER VALUE OR NONE]`
- **Supporting evidence:** `[ENTER VALUE]`

Clearly distinguish Wazuh-assigned mappings from analyst-assessed mappings.

MITRE ATT&CK describes behavior and does not independently prove malicious intent.

## Severity Guidance

### Informational

Use when expected process activity is collected without a security alert or meaningful risk.

### Low

Use when:

- The activity was authorized.
- The command was read-only.
- The process and command line were expected.
- A Wazuh alert required validation but presented limited risk.

### Medium

Use when:

- PowerShell or another interpreter runs without documented authorizationle location.
- Important questions remain after initial review.

### High

Use when:

- Encoded or obfuscated execution is confirmed.
- A malicious process hash is identified.
- A trusted application launches a suspicious interpreter.
- The process establishes persistence.
- Credential access or lateral movement is supported by evidence.
- Unauthorized execution results in system changes.

### Critical

Use when:

- Process activity supports active ransomware.
- Malicious execution affects multiple systems.
- Destructive commands are executed.
- Active data exfiltration or widespread compromise is confirmed.

## Final Disposition Options

Choose one:

- Authorized benign activity
- Authorized benign true positive
- False positive
- Suspicious activity
- Confirmed incident
- Insufficient evidence

Record:

- **Severity:** `[ENTER VALUE]`
- **Confidence:** `[LOW/MEDIUM/HIGH]`
- **Final disposition:** `[ENTER VALUE]`
- **Security incident:** `[YES/NO/UNDETERMINED]`
- **Rationale:** `[ENTER EVIDENCE-BASED EXPLANATION]`

## Response Actions

Potential response actions include:

- No action required
- Continue monitoring
- Validate activity with the user or system owner
- Preserve the process executable or script
- Review the process hash
- Terminate a malicious process
- Quarantine a suspicious file
- Disable a compromised account
- Isolate the endpoint
- Block suspicious network destinations
- Review related process, file, and authentication activity
- Escalate to incident response

Response actions must be proportionate to the available evidence and assessed risk.

## Escalation Criteria

Escalate when:

- The process execution was not authorized.
- The executing user denies the activity.
- The command line is encoded, obfuscated, or destructive.
- The parent-child relationship is suspicious.
- The executable launches from an unexpected location.
- The file hash is known to be malicious.
- Discovery activity is followed by credential access or lateral movement.
- Persistence or defense evasion is supported.
- Similar activity appears across multiple endpoints.
- The activity continues after initial review.
- Severity is Medium, High, or Critical.

## Closure Criteria

Close the alert when:

- Sysmon Event ID `1` has been validated.
- The process image and full command line have been reviewed.
- The parent-child relationship has been assessed.
- User and integrity context have been reviewed.
- The process hash has been examined when available.
- Related process, file, authentication, and network activity has been reviewed.
- Authorization and legitimate explanations have been evaluated.
- MITRE ATT&CK mappings have been reviewed.
- Evidence limitations have been documented.
- Severity and confidence have been assigned.
- The final disposition is supported by evidence.
- Required response actions have been completed or escalated.

## Validation Examples

This playbook was validated through:

- ../detection-scenarios/01-sysmon-process-creation/README.md
- ../detection-scenarios/04-powershell-account-discovery/README.md

### Scenario 1

The controlled Notepad execution demonstrated:

- Sysmon Process Create Event ID `1`
- Process image and command-line review
- Parent-process analysis
- SHA-256 process hashing
- Local-to-Wazuh telemetry validation
- Event-versus-alert classification

The activity was classified as authorized benign activity with Informational severity and High confidence.

### Scenario 4

The controlled PowerShell execution demonstrated:

- A child `powershell.exe` process
- Complete command-line capture
- `Get-LocalUser` account-discovery behavior
- Parent-child process analysis
- High integrity execution
- Wazuh rule `92027`
- Wazuh-assigned `T1059.001`
- Analyst-assessed `T1087.001`

The activity was classified as an authorized benign true positive with Low severity and High confidence.

## Completion Checklist

- [x] Purpose documented
- [x] Trigger conditions documented
- [x] Required Sysmon Event ID `1` fields documented
- [x] Process-image review documented
- [x] Command-line analysis documented
- [x] Parent-child process review documented
- [x] User and integrity analysis documented
- [x] Hash-review procedure documented
- [x] Timeline and scope workflow documented
- [x] Benign explanations documented
- [x] Suspicious indicators documented
- [x] MITRE ATT&CK review documented
- [x] Severity guidance documented
- [x] Response actions documented
- [x] Escalation criteria documented
- [x] Closure criteria documented
- [x] Playbook validated against Scenarios 1 and 4

## Playbook Status

**Status:** Complete

**Primary use:** Investigation of Sysmon Process Create Event ID `1`, PowerShell execution, suspicious command lines, and unusual parent-child process relationships.
