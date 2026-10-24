# Scenario 2: Controlled Failed Windows Logons

## Scenario Objective

Validate the collection and investigation of controlled Windows authentication failures on ENDPOINT-01.

The scenario confirmed whether Windows recorded the failed sign-in attempts as Security Event ID `4625` and whether the Windows Wazuh agent forwarded the corresponding telemetry to SIEM-01.

Event ID `4625` represents an account failing to log on and is generated on the computer where the failed logon attempt occurred.

## Environment

- **Target system:** ENDPOINT-01
- **Operating system:** Windows 11
- **Wazuh agent name:** `windows-target`
- **SIEM:** SIEM-01
- **Test account:** Dedicated local lab account
- **Test location:** Windows lock screen
- **Planned failed attempts:** Two
- **Authorization:** Testing restricted to systems that I own and operate

TEST-01 was not required for this scenario because the controlled authentication failures were generated locally on ENDPOINT-01.

## Safety Controls

Before testing, I reviewed the Windows account-lockout policy and limited the test to two incorrect password attempts.

The purpose was to generate failed-logon telemetry, not to test password strength, trigger account lockout, perform brute force, or simulate password spraying.

Safety requirements included:

- Use only a dedicated lab account.
- Confirm the correct password before locking Windows.
- Review the account-lockout threshold.
- Remain below the configured lockout threshold.
- Stop testing if the account becomes locked.
- Do not record or publish the password.
- Do not test accounts or systems outside the authorized lab.

## Prerequisite Validation

### Prerequisite Evidence

Failed-logon auditing was enabled for successful and failed logon events. The local account-lockout threshold was configured for 10 failed attempts, so I limited the controlled test to two incorrect-password attempts.

![Windows failed-logon auditing and account-lockout settings](01-audit-and-lockout-prerequisites.png)

*Figure 1. Windows Logon auditing was configured for Success and Failure. The account-lockout threshold was 10 attempts, with a 10-minute lockout duration and a 10-minute observation window.*

## Controlled Activity

I generated two controlled incorrect-password attempts against the dedicated local lab account `user01` at the Windows lock screen on ENDPOINT-01.

After the two failed attempts, I signed in using the correct password and stopped the test. The test remained safely below the configured lockout threshold of 10 attempts.

### Test Record

- **Test date:** October 23, 2026
- **Target endpoint:** ENDPOINT-01
- **Target account:** `user01`
- **Test location:** Windows lock screen
- **Failed attempts generated:** Two
- **First failed attempt:** October 23, 2026 at 6:08:06 PM
- **Second failed attempt:** October 23, 2026 at 6:08:08 PM
- **Interval between attempts:** Approximately two seconds
- **Subsequent successful sign-in:** Yes
- **Unexpected account lockout:** No

## Local Windows Evidence

Windows recorded both controlled authentication failures as Security Event ID `4625`.

The selected event represents the second controlled failed sign-in attempt and contains the following evidence:

- **Time created:** October 23, 2026 at 6:08:08 PM
- **Event ID:** `4625`
- **Provider:** `Microsoft-Windows-Security-Auditing`
- **Message:** An account failed to log on
- **Target account:** `user01`
- **Target domain:** `WIN-ENDPOINT`
- **Logon type:** `2`
- **Failure reason:** Unknown user name or bad password
- **Status:** `0xC000006D`
- **Sub-status:** `0xC000006A`
- **Caller process:** `C:\Windows\System32\svchost.exe`
- **Workstation name:** `WIN-ENDPOINT`

Logon Type `2` is consistent with an interactive sign-in attempt at the Windows console, which matches the controlled lock-screen test.

![Local Windows Security Event ID 4625](02-windows-event-4625-local.png)

*Figure 2. Windows Security Event ID 4625 recorded the second of two controlled failed interactive sign-in attempts on ENDPOINT-01. The event identified Logon Type 2, the target account, failure reason, status and sub-status codes, caller process, and workstation.*

## Wazuh Detection Evidence

I searched Wazuh Threat Hunting for Windows Security Event ID `4625` using the following query:

![Wazuh Threat Hunting results for Windows Event ID 4625](03-wazuh-event-4625.png)

*Figure 3. Wazuh Threat Hunting displayed three Event ID 4625 alerts from the monitored Windows endpoint. Rule ID `60122`, at rule level `5`, classified the activity as a logon failure involving an unknown username or incorrect password.*

