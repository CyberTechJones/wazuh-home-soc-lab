# Home SOC Lab System Inventory

**Inventory Overview**

This inventory documents the virtual machines and security components used in my home SOC lab. Sensitive information, including IP addresses, credentials, enrollment keys, tokens, MAC addresses, and other unnecessary system identifiers, has been redacted from the public repository.

**Virtualization Environment**

* Virtualization platform: Oracle VirtualBox
* Environment type: Three-VM home SOC lab
* Virtual network type: Oracle VirtualBox Host-only network with NAT access on selected VMs (see screenshots below)
![Kali Network](./kali-network.png)
![Win Network](./win-network.png)
![Wazuh Network](./wazuh-network.png)

* Internet access during testing: Available to the Wazuh and Kali Linux VMs through separate NAT adapters
* Primary purpose: Defensive security monitoring, detection validation, and SOC investigation practice
---
**Wazuh SIEM**

The Wazuh virtual machine operates as the Security Information and Event Management platform for the lab. It receives and analyzes security telemetry from the monitored endpoints, applies detection rules, and provides centralized alerting, search, visualization, and investigation capabilities.

* Asset ID: SIEM-01
* VM name: wazuh-server
* Hostname: wazuh-server
* IP address: [REDACTED]
* Operating system: ubuntu 24.10
* Wazuh version: Version 4.14
* Primary role: Security Information and Event Management platform
* Current status: Configured

![agent](./agents.png)
![wazuh](./wazuh.png)

**Primary Functions**

* Log collection and ingestion
* Detection rules and alerting
* Search and visualization
* Investigation and reporting
---
**Windows Endpoint**

The Windows virtual machine serves as the primary monitored endpoint. The Wazuh agent collects configured Windows telemetry and forwards it to the Wazuh server for analysis.

* Asset ID: ENDPOINT-01
* VM name: win-endpoint
* Hostname: win-endpoint
* IP address: [REDACTED]
* Operating system: windows 11
* Wazuh agent installed: Yes
* Wazuh agent status: [ACTIVE]
* Wazuh agent version: [VERIFIED]
* Sysmon status: [INSTALLED]
* Primary role: Monitored Windows workstation
* Current status: Configured

**Data Sources**

* Windows Security event log
* Windows System event log
* Windows Application event log
* Wazuh agent telemetry
* Sysmon Operational log, when configured
* File Integrity Monitoring, when configured
---
**Kali Linux Test System**

The Kali Linux virtual machine generates safe, controlled security activity against the Windows endpoint within the isolated lab. The Wazuh agent also monitors the Kali system and forwards configured Linux telemetry to the Wazuh server.

* Asset ID: TEST-01
* VM name: kali-attacker
* Hostname: kali-attacker
* IP address: [REDACTED]
* Operating system: Version 2026.2
* Wazuh agent installed: Yes
* Wazuh agent status: [ACTIVE]
* Wazuh agent version: [VERIFIED]
* Primary role: Authorized security-testing and monitored Linux system
* Current status: Configured

**Primary Functions**

* Generate controlled security activity
* Support Windows detection validation
* Forward configured Linux telemetry to Wazuh
* Support cross-system timeline correlation
* Perform testing only against authorized lab systems
---
**Asset Summary**

![Asset summary table](./asset-summary.png)

**Data Flow Summary**

1. TEST-01 generates controlled and authorized activity against ENDPOINT-01.
2. ENDPOINT-01 records relevant Windows and endpoint activity.
3. The Windows Wazuh agent forwards configured telemetry to SIEM-01.
4. The Kali Linux Wazuh agent forwards configured Linux telemetry to SIEM-01.
5. SIEM-01 analyzes, indexes, and displays the collected security data.
6. The analyst reviews the evidence and determines the appropriate alert disposition.

**Inventory Validation Status**

* Wazuh virtual machine created
* Windows virtual machine created
* Kali Linux virtual machine created
* Windows Wazuh agent installed
* Kali Linux Wazuh agent installed
* Wazuh hostname verified
* Windows hostname verified
* Kali Linux hostname verified
* Operating-system versions documented
* Wazuh version documented
* Windows Wazuh agent shown as active
* Kali Linux Wazuh agent shown as active
* Windows log collection validated
* Kali Linux log collection validated
* Oracle VirtualBox network type documented
* System time synchronization validated across all VMs
* Clean snapshots created for all VMs

**Public Documentation Notice**

This inventory uses Asset IDs and redacted values to describe the lab without exposing unnecessary system information. Actual network addresses, credentials, authentication keys, and other private identifiers are maintained separately and are not included in this repository.






