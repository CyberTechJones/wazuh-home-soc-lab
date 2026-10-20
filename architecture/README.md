Home SOC Lab Architecture
Architecture Overview
This project uses three virtual machines connected through an isolated virtual network. Each virtual machine has a specific role in the detection and investigation workflow.

The environment is designed for authorized defensive cybersecurity testing and portfolio development.

Lab Systems
Wazuh Server
Asset ID: SOC-01
Hostname: WAZUH-SRV
IP address: [WAZUH-IP]

The Wazuh server provides centralized security monitoring for the lab.

Its responsibilities include:

Receiving telemetry from the Windows endpoint
Analyzing collected security events
Applying detection rules
Indexing security data
Displaying alerts and events through the Wazuh dashboard
Supporting investigation and reporting
Windows Endpoint
Asset ID: ENDPOINT-01
Hostname: WIN-ENDPOINT
IP address: [WINDOWS-IP]

The Windows endpoint represents the monitored workstation.

Its responsibilities include:

Generating Windows event logs
Running the Wazuh agent
Forwarding configured telemetry to the Wazuh server
Producing enhanced endpoint telemetry through Sysmon
Providing evidence for detection and investigation scenarios
Kali Linux Test System
Asset ID: TEST-01
Hostname: KALI-TEST
IP address: [KALI-IP]

The Kali Linux virtual machine is the authorized security-testing system.

Its responsibilities include:

Generating controlled security activity
Supporting safe detection validation
Operating only within the isolated virtual lab
Testing systems that I own and am authorized to use
Data Flow
Kali Linux Test System
        |
        | Authorized and controlled test activity
        v
Windows Endpoint
        |
        | Windows logs, Sysmon events, and file changes
        v
Wazuh Agent
        |
        | Collected endpoint telemetry
        v
Wazuh Server
        |
        | Detection, indexing, investigation, and reporting
        v
SOC Analyst Review
