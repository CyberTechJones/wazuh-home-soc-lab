# Scenario 4: PowerShell Local-Account Discovery

## Objective

Validate whether Sysmon and Wazuh provide visibility into controlled PowerShell execution and local-account discovery on ENDPOINT-01.

The scenario uses the read-only `Get-LocalUser` cmdlet to retrieve local Windows account information without modifying accounts, passwords, privileges, or system configuration.

## Environment

- **Target asset:** ENDPOINT-01
- **Operating system:** Windows 11
- **Wazuh agent:** `windows-target`
- **Wazuh agent ID:** `001`
- **SIEM:** SIEM-01
- **Telemetry source:** Sysmon Event ID `1`
- **Controlled command:** `Get-LocalUser`
- **Scenario marker:** `SOC-Lab-Scenario4`
- **Authorization:** Testing restricted to systems that I own and operate

TEST-01 is not required because the controlled PowerShell activity is generated locally on ENDPOINT-01.

## Safety Controls

The scenario uses a read-only PowerShell command.

- Retrieve local-account information only.
- Do not create, delete, enable, or disable accounts.
- Do not change passwords or group memberships.
- Do not export account information outside the lab.
- Keep testing restricted to ENDPOINT-01.
- Exclude passwords, tokens, and unnecessary identifiers from screenshots.
- Preserve only sanitized public evidence.

## Planned Evidence

- PowerShell command availability
- Sysmon and Wazuh service status
- Controlled `Get-LocalUser` results
- Local Sysmon Process Create Event ID `1`
- Full PowerShell command line
- Parent-process information
- Corresponding Wazuh event or alert
- MITRE ATT&CK assessment
- Final event and alert disposition

## Current Status

- [ ] `Get-LocalUser` availability verified
- [ ] Sysmon service confirmed running
- [ ] Wazuh agent confirmed running
- [ ] ENDPOINT-01 confirmed Active
- [ ] Test start time recorded
- [ ] Controlled account-discovery command executed
- [ ] Local Sysmon Event ID `1` located
- [ ] Full command line confirmed
- [ ] Corresponding Wazuh telemetry located
- [ ] Local and centralized evidence correlated
- [ ] MITRE ATT&CK mapping reviewed
- [ ] Final disposition recorded
- [ ] Public evidence reviewed and sanitized

## Scenario Status

**Status:** In progress

**Current finding:** Prerequisite validation and controlled PowerShell testing are pending.


## Prerequisite Validation

Before executing the controlled account-discovery command, I confirmed that `Get-LocalUser` was available and that both Sysmon and the Windows Wazuh agent were running on ENDPOINT-01.

![Scenario 4 prerequisite validation](01-scenario4-prerequisites.png)

*Figure 1. ENDPOINT-01 prerequisite validation confirmed that `Get-LocalUser` was available through the `Microsoft.PowerShell.LocalAccounts` module and that the Sysmon and Wazuh services were running.*
