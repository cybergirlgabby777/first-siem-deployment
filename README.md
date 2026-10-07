# My First SIEM Deployment
 
## Overview
This project documents my first Security Information and Event Management (SIEM) deployment using Wazuh. The project includes integrating Windows Defender logs, validating telemetry collection, and executing Atomic Red Team experiments mapped to the MITRE ATT&CK framework.
 
## Tools Used
- Wazuh
- Windows Defender
- Sysmon
- Atomic Red Team
- MITRE ATT&CK
- PowerShell
 
## Modification
- Added Windows Defender Operational logs as a custom Wazuh log source.
 
## Experiments
1. T1047 - Windows Management Instrumentation (WMI)
2. T1566.001 - Spearphishing Attachment
3. T1569.002 - System Services
 
## Key Findings
- Verified successful Windows Defender log ingestion.
- Validated Wazuh alert generation.
- Observed process creation, service creation, and Defender telemetry.
- Correlated endpoint activity with SIEM events.
 
## Blog Post
See the full project writeup in this repository.
 
## Author
Gabrielle Lankester
