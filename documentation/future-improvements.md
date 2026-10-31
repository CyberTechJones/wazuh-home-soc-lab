# Future Improvements

## Overview

The current Wazuh home SOC lab provides working endpoint monitoring, centralized event collection, controlled detection testing, evidence-based investigations, and reusable SOC playbooks.

Future work will focus on improving Windows telemetry, creating custom detection logic, expanding authentication testing, and addressing documented visibility limitations.

Changes will be introduced incrementally so each improvement can be tested, documented, and rolled back safely.

## Priority Roadmap

| Priority | Improvement | Primary Value | Status |
|---|---|---|---|
| High | Enhanced PowerShell logging | Better script and command visibility | Planned |
| High | Custom Wazuh rule development | Detection-engineering experience | Planned |
| High | Standard investigation template | Consistent case documentation | Planned |
| Medium | Microsoft Defender telemetry | Additional endpoint visibility | Planned |
| Medium | Remote authentication testing | Source and Logon Type analysis | Planned |
| Medium | Event retention improvements | Better historical investigations | Planned |
| Medium | Detection-delay measurement | Quantitative SIEM validation | Planned |
| Future | Suricata integration | Network and endpoint correlation | Future |
| Future | Evidence-collection automation | Faster, consistent documentation | Future |
| Future | Additional endpoints | Multi-host investigation experience | Future |

## 1. Enhanced PowerShell Logging

### Objective

Expand PowerShell visibility beyond Sysmon Process Create Event ID `1`.

### Planned Telemetry

- Script Block Logging, Event ID `4104`
- Module Logging, Event ID `4103`
- PowerShell Operational events
- Sysmon Event ID `1`
- Wazuh event-channel collection

### Validation

1. Enable enhanced PowerShell logging on ENDPOINT-01.
2. Configure the Wazuh agent to collect the relevant event channel.
3. Execute a harmless read-only command.
4. Confirm the event locally and in Wazuh.
5. Correlate PowerShell events with Sysmon timestamps and command lines.

**Success criterion:** Wazuh receives readable PowerShell telemetry that adds context beyond the original process command line.

**Status:** Planned

## 2. Custom Wazuh Detection Rule

### Objective

Create, test, and tune a custom Wazuh rule for repeated Windows failed logons.

### Proposed Test

Evaluate a threshold such as:

```text
Five failed logons involving the same account within two minutes
```

The threshold is a lab test condition, not a production recommendation.

### Validation

1. Define the required Event ID `4625` fields.
2. Create the custom rule.
3. Validate the rule syntax.
4. Test below the threshold.
5. Test at the threshold.
6. Confirm the expected alert behavior.
7. Tune and retest the rule.
8. Document false-positive risks and limitations.

**Success criterion:** Below-threshold activity does not trigger the custom rule, while activity meeting the documented conditions does.

**Status:** Planned

## 3. Standard Investigation Template

### Objective

Create a reusable case record for authentication, process, PowerShell, and FIM investigations.

```text
Case ID:
Alert timestamp:
Asset:
User or account:
Wazuh rule:
Event ID:
MITRE ATT&CK mapping:

Summary:
Evidence reviewed:
Timeline:
Scope:
Evidence proves:
Evidence does not prove:
Evidence limitations:

Severity:
Confidence:
Final disposition:
Response actions:

Analyst conclusion:
```

**Success criterion:** Every alert can be documented consistently without duplicating the full playbooks.

**Status:** Planned

## 4. Microsoft Defender Telemetry

### Objective

Collect useful Microsoft Defender events from ENDPOINT-01 and correlate Defender activity with Sysmon and Wazuh.

### Planned Coverage

- Malware-detection events
- Remediation events
- Protection-status changes
- Scan results
- Configuration or exclusion changes

Testing will use safe validation methods. Live malware will not be introduced.

**Success criterion:** A Defender event is confirmed locally and correlated with the corresponding Wazuh telemetry.

**Status:** Planned

## 5. Remote Authentication Testing

### Objective

Extend Scenario 2 by generating authorized network-based authentication activity from TEST-01.

### Investigation Goals

- Compare local and remote Logon Types
- Review source addresses and ports
- Correlate Windows and Kali timestamps
- Analyze failed and successful authentication sequences
- Test source-based alert scoping

Testing will remain below the account-lockout threshold and within the isolated VirtualBox network.

**Success criterion:** Windows and Wazuh record the remote attempt with enough source context to support investigation.

**Status:** Planned

## 6. Event Retention and Archive Access

### Objective

Improve access to non-alerting events without permanently storing every event.

Scenario 1 showed that Wazuh received a benign Notepad event even though the event did not appear in the alert index. Temporary JSON archiving confirmed collection but increased storage use.

### Planned Improvements

- Review Wazuh archive and retention options
- Evaluate selective archive indexing
- Monitor storage growth
- Define a lab retention period
- Document a safe enable-and-disable procedure

**Success criterion:** Non-alerting events can be retrieved when needed without uncontrolled storage growth.

**Status:** Planned

## 7. Detection-Delay Measurement

### Objective

Measure the delay between endpoint activity and Wazuh visibility.

For each controlled event, record:

1. Local action time
2. Endpoint event time
3. Wazuh alert time
4. Dashboard indexing time

```text
Observed delay = Wazuh timestamp - endpoint timestamp
```

At least five controlled events will be measured, with time zones normalized before comparison.

**Success criterion:** Median and observed delay ranges are documented for Sysmon, authentication, and FIM telemetry.

**Status:** Planned

## 8. Longer-Term Improvements

### Suricata Network Telemetry

Add network-based visibility for DNS, connections, scanning, and suspicious outbound activity. Correlate Suricata alerts with Sysmon, authentication, PowerShell, and FIM evidence.

### Evidence-Collection Automation

Create PowerShell and Linux scripts to collect selected events, service status, hashes, timestamps, and sanitized configuration snippets.

Automation must preserve timestamps, avoid modifying evidence sources, and exclude credentials or tokens.

### Additional Endpoints

Potential additions include:

- A second Windows workstation
- A Windows Server evaluation VM
- An Ubuntu server
- A dedicated network sensor

Additional systems will be introduced only if the current host has sufficient CPU, memory, storage, and snapshot capacity.

**Status:** Future

## Implementation Phases

### Phase 1: Improve Windows Visibility

1. Enable enhanced PowerShell logging.
2. Collect PowerShell events in Wazuh.
3. Measure detection delay.
4. Create the investigation template.

### Phase 2: Begin Detection Engineering

1. Create a custom failed-logon rule.
2. Test below-threshold and at-threshold behavior.
3. Tune the rule.
4. Document false-positive risks.
5. Retest after each change.

### Phase 3: Expand Telemetry

1. Add Microsoft Defender events.
2. Test remote authentication from TEST-01.
3. Improve archive access.
4. Evaluate Suricata integration.

### Phase 4: Scale and Automate

1. Automate evidence collection.
2. Add monitored endpoints.
3. Test multi-host correlation.
4. Develop more advanced incident-response exercises.

## Success Measures

Each improvement should produce measurable evidence:

- Local telemetry confirmed
- Centralized Wazuh telemetry confirmed
- Detection behavior tested
- Delay or storage impact measured
- False-positive conditions documented
- Rollback procedure validated
- Evidence sanitized
- Final conclusion supported by observed results

## Next Milestone

The next major milestone is a custom threshold-based failed-logon rule.

This project is the strongest next step because the lab already includes:

- Windows Event ID `4625` evidence
- A completed failed-logon investigation
- A validated failed-logon playbook
- Known account-lockout settings
- Existing Wazuh authentication alerts
- A safe procedure for generating controlled failures

The custom rule will demonstrate progression from validating default Wazuh behavior to designing, testing, and tuning detection logic.
