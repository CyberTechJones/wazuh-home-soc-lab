# File Integrity Monitoring Investigation Playbook

## Purpose

Provide a repeatable workflow for investigating Wazuh File Integrity Monitoring alerts involving file creation, modification, renaming, or deletion.

This playbook helps an analyst determine whether a file change resulted from authorized administration, software activity, user activity, misconfiguration, or potentially malicious behavior.

## Related Playbooks

Use the general triage process for shared investigation steps:

./alert-triage-playbook.md

## Validation Scenario

This playbook was validated through:

../detection-scenarios/03-file-integrity-monitoring/README.md

The controlled validation tested file creation, modification, renaming, and deletion inside:

```text
C:\SOC-Lab\FIM-Test
```

## Trigger Conditions

Use this playbook when:

- Wazuh generates a File Integrity Monitoring alert.
- A monitored file is created, modified, renamed, or deleted.
- A checksum changes unexpectedly.
- A monitored configuration or application file changes.
- Multiple file changes occur within a short period.
- A file change occurs outside an approved maintenance window.
- The responsible user or process is unknown.
- The change affects a critical or sensitive path.

## Initial Alert Information

> **Usage note:** Copy the following fields into a new investigation record and replace each bracketed placeholder with values from the alert being investigated. Do not permanently replace the placeholders with values from the validation scenario.

Record:

- **Alert timestamp:** `[ENTER VALUE]`
- **Wazuh rule ID:** `[ENTER VALUE]`
- **Rule description:** `[ENTER VALUE]`
- **Rule level:** `[ENTER VALUE]`
- **Agent name:** `[ENTER VALUE]`
- **Agent ID:** `[ENTER VALUE]`
- **Hostname:** `[ENTER VALUE]`
- **File path:** `[ENTER VALUE]`
- **FIM action:** `[ADDED/MODIFIED/DELETED/UNKNOWN]`
- **File size before:** `[ENTER VALUE OR NOT AVAILABLE]`
- **File size after:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Previous hash:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Current hash:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Changed attributes:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Content difference:** `[ENTER VALUE OR NOT AVAILABLE]`
- **User attribution:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Process attribution:** `[ENTER VALUE OR NOT AVAILABLE]`

## Investigation Workflow

### 1. Validate the FIM Event

Confirm that the alert contains a Wazuh `syscheck` event for the expected agent and file path.

Review:

- File path
- FIM action
- Event timestamp
- Wazuh rule ID
- Rule description
- Rule level
- File size
- Previous and current hashes
- Changed fields
- Content differences, when available
- User and process attribution, when available

Do not rely only on the rule description. Compare the description with the actual `syscheck.event` and `syscheck.path` values.

Record:

- **FIM event validated:** `[YES/NO/PARTIALLY]`
- **Path confirmed:** `[YES/NO]`
- **Action confirmed:** `[YES/NO]`
- **Validation notes:** `[ENTER VALUE]`

### 2. Identify the File and Its Purpose

Determine:

- Whether the path is expected to be monitored
- Whether the file is a system, application, configuration, script, executable, document, or temporary file
- Whether the filename and extension are expected
- Whether the file is required for business or system operation
- Whether the file existed before the alert
- Whether the path is writable by standard users

Record:

- **File category:** `[SYSTEM/APPLICATION/CONFIGURATION/SCRIPT/EXECUTABLE/DOCUMENT/TEMPORARY/OTHER]`
- **Expected path:** `[YES/NO/UNKNOWN]`
- **Expected file:** `[YES/NO/UNKNOWN]`
- **Operational purpose:** `[ENTER VALUE OR UNKNOWN]`
- **Criticality:** `[LOW/MEDIUM/HIGH/UNKNOWN]`

### 3. Interpret the File Action

#### Added

An `added` event indicates that a new file appeared in the monitored path.

Investigate:

- Whether the file was expected
- Who or what created it
- Whether the filename or extension is suspicious
- Whether the hash is known
- Whether an installation or update occurred

#### Modified

A `modified` event indicates that a monitored file’s attributes or contents changed.

Investigate:

- Which attributes changed
- Whether the hash changed
- Whether text differences are available
- Whether the modification occurred during approved maintenance
- Whether the file should normally change

#### Deleted

A `deleted` event indicates that a monitored file is no longer present at the original path.

Investigate:

- Whether deletion was authorized
- Whether the file was moved or renamed
- Whether a replacement file appeared
- Whether loss of the file affects operations or security controls

#### Renamed

Wazuh may represent a rename as two closely timed events:

1. The original path is recorded as `deleted`.
2. The new path is recorded as `added`.

Correlate the events using:

- Closely matching timestamps
- Same monitored directory
- Similar filename
- Matching size
- Matching hash, when available
- Known user or process activity

Do not claim a rename unless the related events support that conclusion.

### 4. Review Hash Information

Compare available hashes such as:

- MD5
- SHA-1
- SHA-256

Determine:

- Whether the hash changed
- Whether the new hash matches a known-good version
- Whether the file hash appears in internal threat intelligence
- Whether the hash is associated with approved software
- Whether the file content changed but the filename remained the same

Record:

- **Hash algorithm:** `[ENTER VALUE]`
- **Previous hash:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Current hash:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Known-good comparison completed:** `[YES/NO/NOT AVAILABLE]`
- **Result:** `[MATCH/MISMATCH/UNKNOWN]`

A changed hash proves that file content changed. A changed hash does not independently prove malicious activity.

### 5. Review Available Content Changes

If `report_changes` produced a text difference, review:

- Added lines
- Removed lines
- Changed settings
- Added commands
- Modified paths
- New accounts or permissions
- Disabled security controls
- Suspicious encoded or obfuscated content

Do not publish sensitive configuration content, passwords, tokens, keys, or credentials.

Record:

- **Content difference available:** `[YES/NO]`
- **Relevant change:** `[ENTER VALUE]`
- **Sensitive data present:** `[YES/NO]`
- **Published evidence sanitized:** `[YES/NO/NOT APPLICABLE]`

### 6. Determine User and Process Attribution

Review whether the FIM event contains:

- User information
- Process name
- Process ID
- Parent process
- Audit or who-data fields

Correlate with:

- Sysmon Event ID `1`
- Windows Security events
- PowerShell logs
- Software installation events
- Scheduled-task activity
- Authentication records
- Approved maintenance records

Record:

- **Responsible user identified:** `[YES/NO]`
- **Responsible process identified:** `[YES/NO]`
- **User:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Process:** `[ENTER VALUE OR NOT AVAILABLE]`
- **Supporting evidence:** `[ENTER VALUE]`

If attribution is unavailable, state:

```text
The FIM evidence confirms that the file changed but does not identify the responsible user or process.
```

### 7. Build the File-Change Timeline

Document the chronological sequence:

| Time | File Path | Action | Rule ID | Supporting Evidence |
|---|---|---|---:|---|
| `[TIME]` | `[PATH]` | `[ACTION]` | `[RULE]` | `[SOURCE]` |
| `[TIME]` | `[PATH]` | `[ACTION]` | `[RULE]` | `[SOURCE]` |
| `[TIME]` | `[PATH]` | `[ACTION]` | `[RULE]` | `[SOURCE]` |

Search for:

- Earlier modifications to the same file
- New files in the same directory
- Related deletion and addition events
- Process creation near the file-change time
- Authentication activity involving the same endpoint
- Similar changes on other endpoints

### 8. Determine the Scope

Classify the activity as:

- **Isolated:** One file changed on one endpoint
- **Related sequence:** Multiple changes involving one file or directory
- **Distributed:** Similar file changes occurred across multiple endpoints
- **Unknown:** Evidence is insufficient to determine scope

Record:

- **Files affected:** `[ENTER VALUE]`
- **Directories affected:** `[ENTER VALUE]`
- **Endpoints affected:** `[ENTER VALUE]`
- **Related alerts:** `[ENTER VALUE]`
- **Scope:** `[ISOLATED/RELATED SEQUENCE/DISTRIBUTED/UNKNOWN]`

### 9. Evaluate Benign Explanations

Potential benign explanations include:

- Authorized security testing
- Approved application update
- Administrator configuration change
- User-created document
- Scheduled maintenance
- Software installation or removal
- Temporary-file activity
- File rename or cleanup
- Approved deployment process

Validate the explanation using timestamps, user context, process evidence, maintenance records, and known system activity.

Record:

- **Potential legitimate explanation:** `[ENTER VALUE]`
- **Authorization confirmed:** `[YES/NO/UNKNOWN]`
- **Validation method:** `[ENTER VALUE]`
- **Explanation supported:** `[YES/NO/PARTIALLY]`

### 10. Evaluate Suspicious Indicators

Suspicious indicators include:

- Changes to critical Windows or application files
- Unexpected executable or script creation
- Modification outside an approved maintenance window
- New files in startup or persistence locations
- Security-tool configuration changes
- Changes associated with encoded or obfuscated scripts
- Multiple endpoints showing similar unexpected changes
- File deletion followed by suspicious replacement
- Hash mismatch against a trusted baseline
- Unexplained user or process attribution
- Concurrent suspicious authentication or process alerts

Any single indicator may be insufficient. Evaluate the complete timeline and supporting telemetry.

## Observed Wazuh FIM Rules

The Scenario 3 validation observed:

| File action | Rule ID | Rule level | Description |
|---|---:|---:|---|
| Added | `554` | `5` | File added |
| Modified | `550` | `7` | Integrity checksum changed |
| Deleted | `553` | `7` | File deleted |

Rule information must be confirmed in the current alert rather than assumed from the validation scenario.

## MITRE ATT&CK Assessment

Do not assign an ATT&CK technique solely because a file was created, modified, renamed, or deleted.

A defensible mapping requires additional context connecting the activity to a behavior such as:

- Persistence
- Defense evasion
- File and directory discovery
- Data staging
- Indicator removal
- Malicious configuration modification

Record:

- **Wazuh-assigned mapping:** `[ENTER VALUE OR NOT ASSIGNED]`
- **Analyst-assessed mapping:** `[ENTER VALUE OR NONE]`
- **Supporting evidence:** `[ENTER VALUE]`

MITRE ATT&CK describes behavior and does not independently establish malicious intent.

## Severity Guidance

### Informational

Use when expected file activity is collected but does not require meaningful review.

### Low

Use when:

- The change was authorized.
- The file was inside a dedicated test directory.
- No system or application files were affected.
- The event requires validation but presents limited risk.

### Medium

Use when:

- A file changes unexpectedly.
- Authorization cannot be confirmed.
- A monitored application or configuration file is affected.
- The responsible process or user is unknown.
- Repeated changes require further investigation.

### High

Use when:

- A malicious file change is confirmed.
- A critical system file is replaced.
- Unauthorized persistence is established.
- Security controls are modified or disabled.
- The file activity is associated with compromise indicators.

### Critical

Use when:

- File changes support active ransomware or destructive activity.
- Multiple critical systems are affected.
- The changes create significant operational impact.
- Malicious activity is actively spreading.

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
- Validate the change with the system owner
- Preserve the changed file
- Compare the file with a trusted copy
- Calculate and review file hashes
- Restore the file from a trusted source
- Quarantine a suspicious file
- Stop the responsible process
- Isolate the endpoint
- Review related process and authentication telemetry
- Escalate to incident response

Response actions must be proportionate to the evidence and potential impact.

## Escalation Criteria

Escalate when:

- The file change was not authorized.
- A critical system or application file was affected.
- The responsible user or process cannot be validated.
- A suspicious executable or script was created.
- A file hash does not match the trusted baseline.
- Security controls or configurations were changed.
- Similar activity appears across multiple endpoints.
- The change is associated with persistence or defense evasion.
- The activity continues after initial review.
- Severity is Medium, High, or Critical.

## Closure Criteria

Close the alert when:

- The affected file and path have been identified.
- The FIM action has been validated.
- Relevant hashes and attributes have been reviewed.
- Rename-related events have been correlated, when applicable.
- User and process attribution have been reviewed.
- Related activity and scope have been evaluated.
- Legitimate explanations have been considered.
- Evidence limitations have been documented.
- Severity and confidence have been assigned.
- The final disposition is supported by evidence.
- Required response actions have been completed or escalated.

## Validation Example

This playbook was validated through:

../detection-scenarios/03-file-integrity-monitoring/README.md

The controlled test monitored:

```text
C:\SOC-Lab\FIM-Test
```

The test generated:

```text
Created → Modified → Renamed → Deleted
```

Wazuh recorded:

- Rule `554`, level `5`, for file additions
- Rule `550`, level `7`, for the content modification
- Rule `553`, level `7`, for deletions

The rename appeared as deletion of the original path followed by addition of the renamed path.

The activity was classified as an authorized benign true positive with Low severity and High confidence.

## Completion Checklist

- [x] Purpose documented
- [x] Trigger conditions documented
- [x] Required FIM fields documented
- [x] File-action interpretation documented
- [x] Hash-review procedure documented
- [x] Content-change review documented
- [x] User and process attribution documented
- [x] Timeline and scope workflow documented
- [x] Benign explanations documented
- [x] Suspicious indicators documented
- [x] Severity guidance documented
- [x] Response actions documented
- [x] Escalation criteria documented
- [x] Closure criteria documented
- [x] Playbook validated against Scenario 3

## Playbook Status

**Status:** Complete

**Primary use:** Investigation of Wazuh File Integrity Monitoring alerts involving file creation, modification, rename, and deletion.
