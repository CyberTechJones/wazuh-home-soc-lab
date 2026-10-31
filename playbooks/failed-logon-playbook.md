# Failed Windows Logon Investigation Playbook

## Purpose

Provide a repeatable workflow for investigating Windows failed-authentication alerts involving Security Event ID `4625`.

This playbook helps an analyst determine whether failed logons resulted from user error, outdated credentials, authorized testing, suspicious authentication activity, or a potential credential attack.

## Related Detection Scenario

This playbook was validated through:

../detection-scenarios/02-failed-windows-logons/README.md

## Trigger Conditions

Use this playbook when:

- Wazuh generates a failed-logon alert.
- Windows Security Event ID `4625` is observed.
- Multiple authentication failures involve the same account.
- Multiple accounts are targeted from the same source.
- Failed logons are followed by a successful authentication.
- Authentication failures occur outside expected working patterns.
- The source system, account, or activity cannot be immediately validated.

## Initial Alert Information

Record:

- **Alert timestamp:** `[ENTER VALUE]`
- **Wazuh rule ID:** `[ENTER VALUE]`
- **Rule description:** `[ENTER VALUE]`
- **Rule level:** `[ENTER VALUE]`
- **Agent name:** `[ENTER VALUE]`
- **Hostname:** `[ENTER VALUE]`
- **Windows Event ID:** `[ENTER VALUE]`
- **Target account:** `[ENTER VALUE]`
- **Target domain:** `[ENTER VALUE]`
- **Logon Type:** `[ENTER VALUE]`
- **Failure reason:** `[ENTER VALUE]`
- **Status:** `[ENTER VALUE]`
- **Sub-status:** `[ENTER VALUE]`
- **Source address:** `[ENTER VALUE OR NOT POPULATED]`
- **Source port:** `[ENTER VALUE OR NOT POPULATED]`
- **Workstation name:** `[ENTER VALUE OR NOT POPULATED]`

## Investigation Workflow

### 1. Validate the Windows Event

Confirm that the underlying Windows event is Security Event ID `4625`.

Review:

- Event timestamp
- Target account
- Target domain
- Failure reason
- Status and sub-status
- Logon Type
- Authentication package
- Logon process
- Caller process
- Workstation name
- Source network address
- Source port

Do not rely only on the Wazuh rule description. Compare the alert with the original Windows event fields.

### 2. Interpret the Logon Type

Use the recorded Logon Type to understand how authentication was attempted.

| Logon Type | Meaning | Common Context |
|---:|---|---|
| `2` | Interactive | Local console or Windows sign-in screen |
| `3` | Network | Shared folders, remote resources, or network authentication |
| `4` | Batch | Scheduled tasks |
| `5` | Service | Windows service logon |
| `7` | Unlock | Workstation unlock |
| `8` | Network cleartext | Authentication where credentials are passed in cleartext form |
| `10` | Remote interactive | Remote Desktop or terminal services |
| `11` | Cached interactive | Cached domain authentication |

Record:

- **Observed Logon Type:** `[ENTER VALUE]`
- **Expected for activity:** `[YES/NO/UNKNOWN]`
- **Interpretation:** `[ENTER EXPLANATION]`

Logon Type should be evaluated together with the account, source, process, and test context.

### 3. Interpret Status and Sub-Status Codes

Review the status codes recorded in the event.

Common examples include:

| Code | General meaning |
|---|---|
| `0xC000006D` | Invalid authentication information |
| `0xC000006A` | Incorrect password |
| `0xC0000064` | Account does not exist |
| `0xC000006F` | Logon attempted outside permitted hours |
| `0xC0000070` | Logon attempted from an unauthorized workstation |
| `0xC0000071` | Password expired |
| `0xC0000072` | Account disabled |
| `0xC0000234` | Account locked |

Record the exact values shown by the event. Do not infer a specific failure reason if the recorded fields do not support it.

### 4. Determine Event Frequency

Count the failed attempts involving:

- The same account
- The same source address
- The same workstation
- Other accounts on the same endpoint
- Other endpoints during the same period

Record:

- **Number of failures:** `[ENTER VALUE]`
- **Time window:** `[ENTER VALUE]`
- **Accounts targeted:** `[ENTER VALUE]`
- **Source systems:** `[ENTER VALUE]`
- **Pattern:** `[ISOLATED/REPEATED/DISTRIBUTED/UNKNOWN]`

A small number of failures may indicate a mistyped password. High-volume or repeated failures may require escalation.

### 5. Search for Related Successful Logons

Search for Windows Security Event ID `4624` involving:

- The same account
- The same source
- The same workstation
- A timestamp shortly after the failures

Record:

- **Related successful logon found:** `[YES/NO]`
- **Success timestamp:** `[ENTER VALUE OR NOT APPLICABLE]`
- **Logon Type:** `[ENTER VALUE OR NOT APPLICABLE]`
- **Source address:** `[ENTER VALUE OR NOT APPLICABLE]`
- **Relationship to failures:** `[ENTER EXPLANATION]`

A successful logon following repeated failures may indicate:

- A user eventually entered the correct password
- Stored credentials were corrected
- An unauthorized actor successfully authenticated
- The successful logon was unrelated

Additional context is required before reaching a conclusion.

### 6. Validate the Account

Determine:

- Whether the account exists
- Whether the account is enabled
- Whether the account is local, domain, service, or system-managed
- Whether the account owner expected the activity
- Whether the account has elevated privileges
- Whether the account was locked
- Whether the password was recently changed or expired

Record:

- **Account type:** `[LOCAL/DOMAIN/SERVICE/SYSTEM/UNKNOWN]`
- **Account enabled:** `[YES/NO/UNKNOWN]`
- **Privileged account:** `[YES/NO/UNKNOWN]`
- **Owner validation completed:** `[YES/NO/NOT APPLICABLE]`
- **Account lockout observed:** `[YES/NO]`

### 7. Evaluate Benign Explanations

Potential benign explanations include:

- Mistyped password
- Authorized security testing
- Recently changed password
- Outdated stored credentials
- Scheduled task using an old password
- Service using invalid credentials
- Mapped network resource
- Expected interactive sign-in failure

Validate benign explanations using the event timeline, target account, source information, and known activity.

### 8. Evaluate Suspicious Indicators

Suspicious indicators include:

- High-volume authentication failures
- Multiple accounts targeted by one source
- One account targeted from multiple sources
- Attempts outside expected working patterns
- Unexpected Remote Desktop Logon Type `10`
- Failures followed by a successful unauthorized login
- Disabled, privileged, or dormant accounts targeted
- Activity across multiple endpoints
- Repeated failures after the account owner denies the activity
- Source addresses outside the expected lab or enterprise network

Any single indicator may be insufficient. Evaluate the complete pattern.

## MITRE ATT&CK Assessment

Do not automatically map isolated failed logons to Brute Force.

A mapping to a Brute Force technique should require evidence such as:

- Automated or repeated attempts
- Significant event volume
- Predictable timing intervals
- Multiple targeted accounts
- Multiple password attempts against one account
- Supporting authentication or network evidence

Record:

- **Wazuh-assigned mapping:** `[ENTER VALUE OR NOT ASSIGNED]`
- **Analyst-assessed mapping:** `[ENTER VALUE OR NONE]`
- **Supporting evidence:** `[ENTER VALUE]`

MITRE ATT&CK mappings describe behavior and do not independently prove malicious intent.

## Severity Guidance

### Informational

Use when the event was collected but did not generate an alert or require meaningful investigation.

### Low

Use when:

- A small number of failures occurred
- The activity was authorized
- User error was confirmed
- No unauthorized successful authentication followed

### Medium

Use when:

- Repeated failures lack a confirmed benign explanation
- The account owner cannot validate the activity
- The source is unexpected
- A privileged account is targeted
- Attempts continue over time

### High

Use when:

- Evidence supports credential compromise
- A suspicious successful logon follows repeated failures
- Multiple systems or accounts are affected
- The activity is associated with lateral movement or privilege escalation

### Critical

Use when:

- Widespread credential compromise is confirmed
- Multiple privileged accounts are compromised
- Authentication activity supports active destructive or high-impact intrusion

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
- **Disposition rationale:** `[ENTER EXPLANATION]`

## Response Actions

Potential response actions include:

- No action required
- Continue monitoring
- Validate activity with the account owner
- Reset the account password
- Revoke active sessions
- Disable or restrict the account
- Review multifactor-authentication status
- Identify and isolate the source system
- Block a source address
- Search for related authentication activity
- Escalate to incident response

Response actions must reflect the evidence and assessed risk.

## Escalation Criteria

Escalate when:

- The activity was not authorized.
- The account owner denies the activity.
- A privileged account was targeted.
- Failures originate from an unexpected source.
- A suspicious successful logon follows the failures.
- Multiple accounts or endpoints are affected.
- Failures continue after initial review.
- Evidence supports brute force or password spraying.
- Account compromise cannot be ruled out.
- Severity is assessed as Medium, High, or Critical.

## Closure Criteria

Close the alert when:

- Event ID `4625` has been validated.
- The target account and endpoint have been identified.
- Logon Type and failure codes have been interpreted.
- Event frequency and scope have been reviewed.
- Related successful logons have been checked.
- Legitimate explanations have been evaluated.
- Severity and confidence have been assigned.
- The final disposition is supported by evidence.
- Required response actions have been completed or escalated.

## Validation Scenario

This playbook was validated through:

../detection-scenarios/02-failed-windows-logons/README.md

The controlled test generated three interactive failed logons on ENDPOINT-01.

The representative event contained:

- Windows Security Event ID `4625`
- Interactive Logon Type `2`
- Status `0xC000006D`
- Sub-status `0xC000006A`
- Target account `user01`
- Workstation `WIN-ENDPOINT`

Wazuh generated three corresponding level `5` alerts under rule `60122`.

The activity was classified as an authorized benign true positive with Low severity and High confidence.

## Completion Checklist

- [x] Purpose documented
- [x] Trigger conditions documented
- [x] Required Event ID `4625` fields documented
- [x] Logon Type interpretation included
- [x] Status-code review included
- [x] Event-frequency analysis included
- [x] Related successful-logon review included
- [x] Benign explanations documented
- [x] Suspicious indicators documented
- [x] Severity guidance documented
- [x] Response actions documented
- [x] Escalation criteria documented
- [x] Closure criteria documented
- [x] Playbook validated against Scenario 2

## Playbook Status

**Status:** Complete

**Primary use:** Investigation of Windows failed-authentication alerts and Security Event ID `4625`.
