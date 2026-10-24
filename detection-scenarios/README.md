# Detection Scenario Roadmap

## Overview
This directory documents the controlled security tests performed within my authorized home SOC lab. Each scenario is designed to validate endpoint telemetry, evaluate Wazuh visibility, and develop repeatable SOC investigation procedures.

All testing is non-destructive and restricted to virtual machines that I own and operate within the controlled Oracle VirtualBox lab environment.

## Lab Systems
The detection scenarios use the following systems:

* **TEST-01:** Kali Linux testing and monitored Linux system
* **ENDPOINT-01:** Monitored Windows 11 endpoint
* **SIEM-01:** Wazuh Security Information and Event Management platform

TEST-01 generates controlled security activity against ENDPOINT-01. The Wazuh agents on TEST-01 and ENDPOINT-01 forward configured telemetry to SIEM-01 for centralized analysis and investigation.

## Scenario Methodology
Each completed detection scenario will document:

1. Scenario objective
2. Environment and assumptions
3. Prerequisites
4. Implementation instructions
5. Controlled security test
6. Expected data sources
7. Wazuh detection and evidence
8. SOC investigation workflow
9. MITRE ATT&CK mapping
10. Severity and confidence assessment
11. Final alert disposition
12. Containment and remediation recommendations
13. Evidence and screenshot checklist
14. Lessons learned

## Scenario Status


---
## Scenario 1: Sysmon Process-Creation Validation
### Objective
Validate the complete telemetry path for a controlled process launch on ENDPOINT-01. The scenario confirmed that Sysmon recorded the process locally and that the Windows Wazuh agent forwarded the corresponding event to SIEM-01.

### Controlled Activity
Launched Windows Notepad on ENDPOINT-01 using the unique command-line marker `C:\SOC-Lab-Scenario1.txt` and reviewed the resulting Sysmon Process Create event.

### Data Sources
* `Microsoft-Windows-Sysmon/Operational`
* Sysmon Event ID `1`
* Windows Wazuh agent telemetry
* Wazuh JSON event archive

### Evidence of Interest
* Event timestamp
* Process image
* Command line
* Current directory
* User context
* Integrity level
* SHA-256 hash
* Parent process
* Parent command line
* Process GUID
* Wazuh agent name
* Wazuh event or alert information

### Skills Demonstrated
* Sysmon Event ID 1 analysis
* Windows process analysis
* Command-line review
* Parent-process analysis
* File-hash review
* Endpoint-to-SIEM telemetry validation
* Timeline correlation
* Event-versus-alert classification

### Current Status
- [x] Sysmon installed on ENDPOINT-01
- [x] Sysmon service confirmed running
- [x] Sysmon Operational event channel confirmed
- [x] Controlled Notepad process launch completed
- [x] Sysmon Event ID 1 validated locally
- [x] Sysmon Operational channel configured in the Windows Wazuh agent
- [x] Windows Wazuh agent remained Active
- [x] Sysmon event collection confirmed in Wazuh
- [x] Exact Notepad event located in the Wazuh JSON archive
- [x] Local and centralized evidence correlated
- [x] Investigation findings documented
- [x] Final disposition recorded
- [x] Temporary JSON event archiving disabled

**Status:** Complete

**Final disposition:** Authorized benign activity with informational severity and high confidence. The activity was not classified as a security incident.

[View the completed Scenario 1 report](./01-sysmon-process-creation/README.md)

## Scenario 2: Repeated Failed Windows Logons
### Objective
Generate and investigate controlled Windows authentication failures to practice account analysis, event correlation, and alert classification.

### Controlled Activity
Intentionally enter an incorrect password several times for a dedicated lab account on ENDPOINT-01.

### Expected Data Source
* Windows Security event log
* Windows Security Event ID 4625
* Windows Wazuh agent telemetry

### Evidence of Interest
* Event timestamp
* Target account
* Workstation name
* Logon type
* Status and sub-status
* Authentication package
* Source address, when populated
* Source port, when populated
* Number and timing of failed attempts
* Related successful authentication, when applicable

### Skills Demonstrated
* Windows authentication analysis
* Failed-logon investigation
* Account and source analysis
* Timeline development
* False-positive analysis
* Severity classification
* Authorized benign true-positive disposition

### Current Status
* Prerequisites validated
* Controlled test completed
* Windows event located
* Wazuh telemetry located
* Investigation completed
* Final disposition documented

**Status:** Complete

**Final disposition:** Authorized benign true positive with Low severity and High confidence. The activity was not classified as a security incident.

[View the completed Scenario 2 report](./02-failed-windows-logons/README.md)

# Scenario 3: File Integrity Monitoring
## Objective
Validate the detection and investigation of controlled file creation, modification, renaming, and deletion within a dedicated Windows test directory.

## Controlled Activity
Use a harmless text file inside:

C:\SOC-Lab\FIM-Test
