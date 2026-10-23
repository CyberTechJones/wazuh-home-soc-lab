# Scenario 1: Sysmon Process-Creation Validation
## Local Sysmon Evidence

I generated a controlled Notepad process launch on ENDPOINT-01 using the unique command-line marker C:\SOC-Lab-Scenario1.txt. Sysmon recorded the activity as Process Create Event ID 1 and captured the executable path, command line, integrity level, SHA-256 hash, and parent-process information.

![Local Sysmon Event ID 1 showing the controlled Notepad process launch](01-sysmon-notepad-local.png)


*Figure 1. Sysmon Event ID 1 recorded the controlled Notepad process launch on ENDPOINT-01. The unique command-line marker SOC-Lab-Scenario1.txt supports correlation with the centralized Wazuh archive record.*
