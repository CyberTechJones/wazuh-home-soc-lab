# Scenario 2: Controlled Failed Windows Logons

## Scenario Objective

Validate the collection and investigation of controlled Windows authentication failures on ENDPOINT-01.

The scenario will confirm whether Windows records the failed sign-in attempts as Security Event ID `4625` and whether the Windows Wazuh agent forwards the corresponding telemetry to SIEM-01.

Microsoft documents Event ID `4625` as **“An account failed to log on.”** The event is generated on the computer where the failed logon attempt occurred. 【1-fbeb62】

## Environment

- **Target system:** ENDPOINT-01
- **Operating system:** Windows 11
- **Wazuh agent name:** `windows-target`
- **SIEM:** SIEM-01
- **Test account:** Dedicated local lab account
- **Test location:** Windows sign-in or lock screen
- **Planned failed attempts:** Two
- **Authorization:** Testing restricted to systems that I own and operate

TEST-01 was not required for this scenario because the failed authentication attempts were generated locally on ENDPOINT-01.

## Safety Controls

Before testing, I reviewed the Windows account-lockout policy and limited the test to two incorrect password attempts.

The purpose was to generate readable failed-logon telemetry, not to test password strength, account lockout, brute force, or password spraying.

Safety requirements:

- Use only a dedicated lab account.
- Confirm the correct password before locking Windows.
- Review the account-lockout threshold.
- Remain below the configured lockout threshold.
- Stop testing if the account becomes locked.
- Do not record or publish the password.
- Do not test accounts or systems outside the authorized lab.

## Prerequisite Validation
### Prerequisite Evidence

Failed-logon auditing was enabled for both successful and failed logon events. The local account-lockout threshold was configured for 10 failed attempts, so I limited the controlled test to two incorrect password attempts.

![Windows failed-logon audit and account-lockout settings](01-audit-and-lockout-prerequisites.png)

*Figure 1. Windows Logon auditing was configured for Success and Failure. The account-lockout threshold was 10 attempts, with a 10-minute lockout duration and 10-minute observation window.*



### Failed-Logon Auditing

I reviewed the Windows Logon audit policy using:

```powershell
auditpol /get /subcategory:"Logon"
