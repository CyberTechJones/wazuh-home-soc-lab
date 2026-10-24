# Scenario 2: Controlled Failed Windows Logons

## Executive Summary

I generated three controlled failed sign-in attempts against a dedicated Windows lab account to validate authentication monitoring in my home SOC environment.

Windows recorded the activity as Security Event ID `4625`, and Wazuh generated three corresponding level `5` alerts under rule `60122`. I reviewed the account context, logon type, failure codes, event volume, and test authorization before classifying the alerts as authorized benign true positives.

## Scenario Objective

This scenario was designed to demonstrate:

- Windows authentication-event analysis
- Wazuh alert validation
- Endpoint-to-SIEM event correlation
- Status and sub-status code review
- Account-lockout safety checks
- Alert severity and confidence assessment
- Event, alert, and incident classification

## Lab Environment

- **Target asset:** ENDPOINT-01
- **Operating system:** Windows 11
- **Wazuh agent:** `windows-target`
- **Wazuh agent ID:** `001`
- **SIEM:** SIEM-01
- **Test account:** Dedicated local lab account
- **Test location:** Windows lock screen
- **Controlled failures:** Three

TEST-01 was not required because the authentication failures were generated locally on ENDPOINT-01.

## Safety Controls

Before testing, I confirmed that the account-lockout threshold was 10 attempts and limited the test to three failed logons.

The test was intended to validate authentication telemetry, not to trigger account lockout, test password strength, or simulate brute force.

Safety controls included:

- Using only a dedicated lab account
- Confirming the correct password before testing
- Remaining below the lockout threshold
- Stopping after three failed attempts
- Excluding the password from all documentation
- Restricting activity to the authorized lab

## Prerequisite Validation

### Logon Auditing

I checked the Windows Logon audit policy with:

```powershell
auditpol /get /subcategory:"Logon"
```

- **Observed result:** Success and Failure
- **Validation:** Pass

### Account-Lockout Policy

I reviewed the local account policy with:

```powershell
net accounts
```

- **Lockout threshold:** 10 attempts
- **Lockout duration:** 10 minutes
- **Observation window:** 10 minutes
- **Planned failures:** Three
- **Validation:** Pass

### Wazuh Agent Health

Before testing, I confirmed that the Wazuh agent was running and that ENDPOINT-01 appeared as Active in the dashboard.

- **Service status:** Running
- **Dashboard status:** Active
- **Validation:** Pass

### Prerequisite Evidence

![Windows failed-logon auditing and account-lockout settings](01-audit-and-lockout-prerequisites.png)

*Figure 1. Windows Logon auditing was enabled for successful and failed activity. The account-lockout threshold was 10 attempts, with a 10-minute duration and observation window.*

## Controlled Test

I entered an incorrect password three times for the dedicated account `user01` at the Windows lock screen.

I then signed in with the correct password and stopped the test.

- **Test date:** October 23, 2026
- **Endpoint:** ENDPOINT-01
- **Target account:** `user01`
- **Failed attempts:** Three
- **Successful sign-in afterward:** Yes
- **Account locked:** No

## Local Windows Evidence

Windows recorded the authentication failures as Security Event ID `4625`.

The representative event contained:

- **Time created:** October 23, 2026 at 6:08:08 PM
- **Event ID:** `4625`
- **Provider:** `Microsoft-Windows-Security-Auditing`
- **Target account:** `user01`
- **Target domain:** `WIN-ENDPOINT`
- **Logon type:** `2`
- **Failure reason:** Unknown user name or bad password
- **Status:** `0xC000006D`
- **Sub-status:** `0xC000006A`
- **Caller process:** `C:\Windows\System32\svchost.exe`
- **Workstation:** `WIN-ENDPOINT`

Logon Type `2` was consistent with an interactive sign-in at the Windows console.

![Local Windows Security Event ID 4625](02-windows-event-4625-local.png)

*Figure 2. Windows Security Event ID 4625 recorded one of three controlled failed interactive sign-in attempts on ENDPOINT-01.*

## Wazuh Detection Evidence

I searched Wazuh Threat Hunting with:

```text
data.win.system.eventID:4625
```

Wazuh displayed three corresponding alerts:

- **Agent:** `windows-target`
- **Agent ID:** `001`
- **Event ID:** `4625`
- **Rule ID:** `60122`
- **Rule level:** `5`
- **Rule description:** Logon Failure - Unknown user or bad password
- **Alert count:** Three

![Wazuh Threat Hunting results for Windows Event ID 4625](03-wazuh-event-4625.png)

*Figure 3. Wazuh displayed three Event ID 4625 alerts. Rule `60122`, at level `5`, identified failed authentication involving an unknown username or incorrect password.*

## Investigation and Correlation

I compared the Windows event with the Wazuh alerts using the following fields:

- Endpoint and Wazuh agent
- Event ID
- Target account and domain
- Interactive Logon Type `2`
- Failure reason
- Status and sub-status codes
- Workstation
- Number of controlled attempts
- Number of Wazuh alerts
- Test window

Three intentional failures produced three Wazuh alerts. The matching account context, Event ID, failure details, alert count, and test window confirmed that Wazuh detected the controlled activity.

## Analyst Assessment

### Evidence Proves

- Three controlled failed sign-ins occurred on ENDPOINT-01.
- Windows recorded the failures as Event ID `4625`.
- The attempts targeted `user01`.
- The representative event used interactive Logon Type `2`.
- Wazuh generated three level `5` alerts.
- Wazuh rule `60122` correctly identified the failed-logon behavior.
- The alert count matched the controlled test count.

### Evidence Does Not Prove

- A brute-force attack occurred.
- Password spraying occurred.
- Credentials were stolen.
- The account was compromised.
- An unauthorized sign-in succeeded.
- Malicious intent existed.

## Event, Alert, and Incident Classification

Windows recorded each failed authentication attempt as an **event**.

Wazuh rule `60122` matched the failed-logon behavior and generated an **alert** for each attempt.

The alerts were **true positives** because the detection correctly identified the failed authentications. The underlying activity was **benign** because the attempts were authorized and intentionally generated.

The activity was not classified as a security incident.

## MITRE ATT&CK Assessment

No MITRE ATT&CK technique was assigned.

Three manual failed-logon attempts were not sufficient evidence of Brute Force behavior. A defensible Brute Force mapping would require stronger indicators such as automation, substantially higher volume, repeated attack patterns, multiple targeted accounts, or other evidence of adversary activity.

Avoiding an unsupported ATT&CK mapping preserved the accuracy of the investigation.

## Final Disposition

- **Activity:** Three controlled failed Windows logons
- **Classification:** Authorized benign true positive
- **Severity:** Low
- **Confidence:** High
- **Security incident:** No
- **Escalation:** Not required
- **Containment:** Not required
- **Remediation:** Not required

The Low severity reflects that Wazuh generated level `5` alerts requiring analyst validation. Authorization, limited volume, the absence of account lockout, and the subsequent successful sign-in reduced the security concern.

## Response Considerations

No response action was required because:

- The activity matched the authorized test plan.
- The expected lab account was targeted.
- The attempts remained below the lockout threshold.
- The account was not locked.
- The correct password remained functional.
- No unauthorized successful access was identified.

For similar unauthorized activity, an analyst should:

1. Validate the activity with the account owner.
2. Review related failed and successful logons.
3. Identify the source system.
4. Determine whether the attempts continued.
5. Reset credentials if compromise is suspected.
6. Restrict the account if credible risk exists.
7. Review lockout and multifactor-authentication controls.
8. Increase monitoring for additional attempts.

## Cleanup

The test did not modify accounts, passwords, files, or system settings.

After testing, I confirmed that:

- The correct password remained functional.
- The account was not locked.
- The Wazuh agent remained Active.
- Failed-logon auditing remained enabled.
- The evidence was preserved and sanitized.

## Skills Demonstrated

- Windows Security event analysis
- Wazuh threat hunting
- Authentication-alert triage
- Status and sub-status code interpretation
- Endpoint-to-SIEM event correlation
- Account-lockout risk management
- False-positive and benign-activity analysis
- Severity and confidence assessment
- Evidence-based incident classification
- Technical documentation

## Lessons Learned

- Event ID `4625` confirms failed authentication, not malicious intent.
- Logon Type `2` represents an interactive console sign-in.
- Status and sub-status codes provide important authentication context.
- Lockout policies should be reviewed before controlled testing.
- A small number of failed logons should not automatically be classified as brute force.
- A detection can be accurate while the underlying activity remains authorized and benign.
- Alert counts should be compared with the documented test record.

## Completion Checklist

- [x] Failed-logon auditing verified
- [x] Account-lockout threshold reviewed
- [x] Three controlled failed logons generated
- [x] Windows Event ID `4625` located locally
- [x] Relevant authentication fields reviewed
- [x] Three corresponding Wazuh alerts located
- [x] Alert count matched the test count
- [x] Wazuh rule ID, level, and description recorded
- [x] Local and centralized evidence correlated
- [x] Final disposition recorded
- [x] Public evidence reviewed and sanitized

## Scenario Status

**Status:** Complete

**Final finding:** Windows recorded three controlled failed sign-in attempts as Event ID `4625`. Wazuh generated three corresponding level `5` alerts under rule `60122`. The activity was classified as an authorized benign true positive with Low severity and High confidence.
