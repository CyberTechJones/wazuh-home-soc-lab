# Lessons Learned

## Overview

Building this authorized Wazuh home SOC lab provided practical experience with endpoint telemetry, centralized event collection, alert investigation, evidence correlation, and incident classification.

The project also demonstrated that successful security monitoring depends on more than installing a SIEM. Each data source must be configured, validated locally, confirmed centrally, and tested through controlled activity.

## Telemetry Collection and Alert Generation

A collected event does not automatically generate an alert.

During the Sysmon Process-Creation scenario, Wazuh received the controlled Notepad event, but the event did not appear in the Threat Hunting alert index because the activity did not match an alert-generating rule. The event was confirmed through the Wazuh JSON archive.

This demonstrated the difference between:

- Endpoint telemetry generation
- Agent collection
- Centralized event receipt
- Detection-rule matching
- Alert generation

## Local and Centralized Correlation

Local endpoint evidence should be compared with centralized SIEM evidence before reaching a conclusion.

Unique command-line markers such as:

```text
SOC-Lab-Scenario4-Retest
