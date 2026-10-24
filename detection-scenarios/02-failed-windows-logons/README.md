# Scenario 2: Controlled Failed Windows Logons

## Objective

Validate whether controlled Windows authentication failures are recorded on ENDPOINT-01 and forwarded to SIEM-01 by the Wazuh agent.

Windows records failed authentication attempts as Security Event ID `4625`.

## Environment

- **Target:** ENDPOINT-01
- **Operating system:** Windows 11
- **Wazuh agent:** `windows-target`
- **Agent ID:** `001`
- **SIEM:** SIEM-01
- **Test account:** Dedicated local lab account
- **Test location:** Windows lock screen
- **Failed attempts:** Three
- **Authorization:** Testing limited to systems I own and operate

TEST-01 was not required because the authentication failures were generated locally on ENDPOINT-01.

## Safety Controls

Before testing, I reviewed the account-lockout policy and confirmed that three failed attempts would remain below the threshold of 10.

The test was designed to generate authentication telemetry, not to trigger account lockout or simulate brute force.

Safety controls included:

- Use only a dedicated lab account.
- Verify the correct password before testing.
- Remain below the lockout threshold.
- Stop if the account becomes locked.
- Do not record or publish the password.
- Restrict testing to the authorized lab.

## Prerequisite Validation

### Logon Auditing

I reviewed the Windows Logon audit policy using:

```powershell
auditpol /get /subcategory:"Logon"
```

- **Result:** Success and Failure
- **Status:** Pass

### Account-Lockout Policy

I reviewed the local account policy using:

```powershell
net accounts
```

- **Lockout threshold:** 10 attempts
- **Lockout duration:** 10 minutes
- **Observation window:** 10 minutes
- **Planned failed attempts:** Three
- **Status:** Pass

### Wazuh Agent

Before testing, I confirmed that the Wazuh agent was running and ENDPOINT-01 appeared as Active in Wazuh.

- **Service status:** Running
- **Dashboard status:** Active
- **Status:** Pass

### Prerequisite Evidence

![Windows failed-logon auditing and account-lockout settings](01-audit-and-lockout-prerequisites.png)

*Figure 1. Windows Logon auditing was enabled for Success and Failure. The lockout threshold was 10 attempts, with a 10-minute duration and observation window.*

## Controlled Activity

I entered an incorrect password three times for the dedicated local account `user01` at the Windows lock screen.

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

Logon Type `2` is consistent with an interactive sign-in at the Windows console.

![Local Windows Security Event ID 4625](02-windows-event-4625-local.png)

*Figure 2. Windows Security Event ID 4625 recorded one of three controlled failed interactive sign-in attempts on ENDPOINT-01.*

## Wazuh Detection Evidence

I searched Wazuh Threat Hunting using:

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

*Figure 3. Wazuh displayed three Event ID 4625 alerts. Rule `60122`, at level `5`, classified the activity as failed authentication involving an unknown username or incorrect password.*

## Investigation and Correlation

The following fields were consistent across the Windows and Wazuh evidence:

- **Endpoint:** ENDPOINT-01
- **Agent:** `windows-target`
- **Event ID:** `4625`
- **Target account:** `user01`
- **Target domain:** `WIN-ENDPOINT`
- **Logon type:** `2`
- **Failure reason:** Unknown user name or bad password
- **Status:** `0xC000006D`
- **Sub-status:** `0xC000006A`
- **Workstation:** `WIN-ENDPOINT`
- **Wazuh rule:** `60122`
- **Controlled failures:** Three
- **Wazuh alerts:** Three

The matching endpoint, account, Event ID, failure details, test window, and alert count confirmed that Wazuh detected the controlled activity.

## Investigation Workflow

I reviewed:

1. The affected endpoint and account
2. The number and timing of failures
3. The logon type
4. The failure reason
5. The status and sub-status codes
6. The caller process and workstation
7. The Wazuh rule information
8. The subsequent successful sign-in
9. Related failures involving other accounts or endpoints

Potential benign causes of Event ID `4625` include mistyped passwords, outdated stored credentials, scheduled tasks, mapped resources, and authorized testing.

The documented test context, Logon Type `2`, limited volume, and matching alert count supported an authorized benign explanation.

## Findings

### Evidence Proves

- Three controlled failed sign-ins occurred on ENDPOINT-01.
- Windows recorded the failures as Event ID `4625`.
- The attempts targeted `user01`.
- The representative event used interactive Logon Type `2`.
- Wazuh generated three rule-level `5` alerts.
- The Wazuh alert count matched the test count.

### Evidence Does Not Prove

- Brute force occurred.
- Password spraying occurred.
- Credentials were stolen.
- The account was compromised.
- Unauthorized access succeeded.
- Malicious intent existed.

## Event and Alert Classification

Windows recorded each failed authentication attempt as an event. Wazuh rule `60122` matched the activity and generated alerts.

The detection was a **true positive** because Wazuh correctly identified the failed-logon behavior. The underlying activity was **benign** because it was authorized and intentionally generated.

No security incident occurred.

## MITRE ATT&CK Mapping

No MITRE ATT&CK technique was assigned.

Three manual failed-logon attempts do not provide sufficient evidence of Brute Force behavior. A Brute Force mapping would require stronger indicators such as automation, higher volume, repeated patterns, or multiple targeted accounts.

## Final Disposition

- **Activity:** Three controlled failed Windows logons
- **Classification:** Authorized benign true positive
- **Severity:** Low
- **Confidence:** High
- **Security incident:** No
- **Escalation required:** No
- **Containment required:** No
- **Remediation required:** No

The Low severity reflects that Wazuh generated rule-level `5` alerts requiring analyst review. Authorization, limited volume, and the successful subsequent sign-in reduced the security concern.

## Response Considerations

No containment or remediation was required because:

- The activity matched the test plan.
- The expected lab account was targeted.
- The attempts remained below the lockout threshold.
- The account was not locked.
- The correct password remained functional.
- No unauthorized successful access was identified.

For similar unauthorized activity, an analyst should:

1. Validate the activity with the account owner.
2. Review related failed and successful logons.
3. Identify the source system.
4. Determine whether attempts continued.
5. Reset credentials if compromise is suspected.
6. Restrict the account if credible risk exists.
7. Review lockout and multifactor-authentication controls.
8. Increase monitoring for additional attempts.

## Cleanup

The test did not modify accounts, passwords, files, or system settings.

I confirmed that:

- The correct password remained functional.
- The account was not locked.
- The Wazuh agent remained Active.
- Failed-logon auditing remained enabled.
- The evidence was preserved and sanitized.

## Lessons Learned

- Event ID `4625` confirms failed authentication, not malicious intent.
- Logon Type `2` represents an interactive console sign-in.
- Status and sub-status codes provide important authentication context.
- Lockout policies should be reviewed before testing.
- A small number of failures should not automatically be classified as brute force.
- A detection can be accurate while the underlying activity remains authorized and benign.
- Alert counts should be compared with the test record.

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

**Final finding:** Windows recorded the three controlled failed sign-in attempts as Event ID `4625`. Wazuh generated three corresponding level `5` alerts under rule `60122`. The activity was classified as an authorized benign true positive with Low severity and High confidence.
