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

### Failed-Logon Auditing

I reviewed the Windows Logon audit policy using:

```powershell
auditpol /get /subcategory:"Logon"
