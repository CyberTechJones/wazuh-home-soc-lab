# Lessons Learned

## Overview

Building this authorized Wazuh home SOC lab provided practical experience with endpoint telemetry, SIEM event collection, alert investigation, evidence correlation, and security-event classification.

The project demonstrated that effective monitoring requires more than installing a SIEM. Each data source must be configured, validated locally, confirmed centrally, and tested through controlled activity.

## Event Collection and Alert Generation

A collected event does not automatically generate an alert.

In Scenario 1, Sysmon recorded the controlled Notepad process and the Wazuh agent forwarded the event to SIEM-01. The event did not appear in Threat Hunting because it did not match an alert-generating rule. Temporary Wazuh JSON archiving confirmed that the event was received.

This demonstrated the difference between:

1. Endpoint telemetry generation
2. Agent collection
3. Centralized event receipt
4. Detection-rule evaluation
5. Alert generation

The absence of an alert does not necessarily indicate a collection failure.

## Evidence Correlation

Local endpoint evidence should be compared with centralized Wazuh evidence before reaching a conclusion.

Unique markers such as:

```text
SOC-Lab-Scenario1.txt
SOC-Lab-Scenario4-Retest
