Moving into cybersecurity wasn't just a career decision for me. It was personal. Coming from a healthcare background, I’ve seen firsthand how catastrophic data breaches can be for organizations and individuals. After witnessing those impacts and navigating a personal hacking incident, I decided to take control and learn how to defend systems myself.

Starting out, typing commands into a terminal felt incredibly daunting. But as I dove into this first SIEM deployment project, something shifted. With practice, using the command line became second nature, and navigating through various software systems felt far less intimidating. In fact, it has completely changed how I understand and interact with my own computer on a daily basis. If you’re just starting out in security, stick with it - you can absolutely get through this!

# Setup

For this deployment, I wanted to expand the monitoring capabilities of my baseline SIEM setup by configuring OSSEC/Wazuh to collect logs directly from **Windows Defender**.

To accomplish this, I modified the `ossec.conf` file on the target endpoint. Adding a custom `<localfile>` block configured for Windows Defender's Event Channel allows the agent to ingest alerts whenever Defender flags suspicious activity.

Here is the exact XML block I added to `ossec.conf`:

```xml

<localfile>

<location>Microsoft-Windows-Windows Defender/Operational</location>

<log_format>eventchannel</log_format>

</localfile>

```

After updating the file, I restarted the Wazuh Agent service to begin log collection.

##### Watch Out For This Common Mistake!

One mistake I made during the setup process was forgetting to restart the Wazuh agent after modifying the `ossec.conf` file. Initially, I expected Windows Defender logs to begin appearing in Wazuh immediately after saving the configuration changes. When no new events appeared, I spent time reviewing the XML configuration before realizing the agent had not been restarted. After restarting the service, the new log source loaded successfully and Windows Defender events began forwarding to Wazuh as expected.

# Experiment Time!
To test how well my SIEM captured security events, I executed three specific test scenarios using **Atomic Red Team** via PowerShell. I logged execution timestamps to track when each technique was triggered and cleaned up afterward.

#### Experiment 1: Execution via WMI (T1047-1)
Invoke-AtomicTest T1047-1 -Session $sess -GetPrereqs
Invoke-AtomicTest T1047-1 -Session $sess
Invoke-AtomicTest T1047-1 -cleanup -Session $sess

#### Experiment 2: Spearphishing Attachment Simulation (T1566.001-1)
Invoke-AtomicTest T1566.001-1 -Session $sess -GetPrereqs
Invoke-AtomicTest T1566.001-1 -Session $sess
Invoke-AtomicTest T1566.001-1 -cleanup -Session $sess

#### Experiment 3: System Service Execution (T1569.002-6)
Invoke-AtomicTest T1569.002-6 -Session $sess -GetPrereqs
Invoke-AtomicTest T1569.002-6 -Session $sess
Invoke-AtomicTest T1569.002-6 -cleanup -Session $sess

## Results

#### Experiment #1
**WMI Execution (`T1047-1`):** Simulates adversary execution using Windows Management Instrumentation.
    - _SIEM Observation:_ Generated process creation logs for `wmic.exe` along with associated process lineage in the dashboard. 

#### Experiment #2
**Spearphishing Attachment (`T1566.001-1`):** Simulates malicious attachment execution patterns.
    - _SIEM Observation:_ Generated process creation and security events that were successfully ingested into Wazuh.

#### Experiment #3
**Service Execution (`T1569.002-6`):** Simulates adversary creation or manipulation of Windows services for code execution.
    - _SIEM Observation:_ Logged Service Control Manager event IDs (`7045` / `7036`) capturing new service creations.


##### Watch Out For This Mistake!

One challenge I encountered during the experimentation phase was keeping track of the different workstations involved in the lab. Since the project required switching between the ART Workstation, Blue-Team Workstation, and the ad01 endpoint, I occasionally became confused about which system was responsible for a particular task. At one point I found myself looking for Wazuh alerts on the ART Workstation and attempting actions from the wrong machine. To avoid repeating this mistake, I began double-checking the workstation hostname and purpose before running commands or reviewing logs.

# Summary of Experimental Findings
The three Atomic Red Team experiments demonstrated how different attack techniques generate distinct telemetry that can be captured and analyzed within a SIEM. The T1047-1 experiment confirmed that WMI activity generated process creation events and command-line artifacts that could be traced through Wazuh. The T1566.001-1 spearphishing simulation successfully triggered Windows Defender detections, validating that my Windows Defender log source modification was working correctly and forwarding relevant security events into the SIEM. Finally, the T1569.002-6 experiment generated Service Control Manager events that highlighted Wazuh's ability to detect and monitor suspicious service creation activity.

Overall, these experiments confirmed that the SIEM was successfully ingesting logs from multiple sources, correlating events, and providing visibility into attacker techniques mapped to the MITRE ATT&CK framework. By comparing activity on the endpoint with the alerts and logs appearing in Wazuh, I was able to verify that the security controls were functioning as expected and identify the types of telemetry generated by each attack technique.

These experiments demonstrated the value of validating detections through controlled adversary emulation rather than simply assuming that log sources are working correctly.


##### Advice on avoiding mistakes
One of the biggest lessons I learned was the importance of maintaining awareness of which workstation I was actively using during testing. Pay attention, or bad actors can slip through the cracks!


# The coolest thing I learned
Beyond demystifying the CLI, the most rewarding revelation was watching raw, isolated event logs from Windows Defender automatically transform into structured, searchable events in a centralized dashboard.

# One piece of advice
Don't let the command line intimidate you. Terminal navigation is a muscle—the more you run commands, verify paths, and break (then fix) configurations, the more intuitive it becomes.

# My favorite resource
The MITRE attack website will be your best friend in times of cyber trouble!
https://attack.mitre.org/
This resource explained how adversaries use WMI for execution and system management activities. It helped me understand the expected behaviors and telemetry generated during the T1047 experiment.

# Thank you (gratitudes)!
Thank you to Rick Rhaburn for your patience and guidance while tutoring me through this Sprint. 

I would also like to thank the TripleTen instructional team for developing the labs and learning materials that made it possible to gain hands-on experience with SIEM deployment and threat detection.


# References
### 1.
**Red Canary (2023). _Atomic Red Team Repository_.** _Description:_ Open-source library of simple, automation-friendly cyber attacks mapped to MITRE ATT&CK. Used to select and execute test procedures `T1047`, `T1566.001`, and `T1569.002`.

### 2.
**Wazuh Documentation Team (2024). _Monitoring Windows Event Logs_.** _Description:_ Official guidance for ingesting Windows Defender Operational channels into Wazuh agents. Used to construct the `ossec.conf` XML configuration block.

### 3.
**MITRE ATT&CK Framework (2024). _Technique T1047: Windows Management Instrumentation_.** _Description:_ Knowledge base detailing adversary use of WMI. Helped identify expected log signatures during testing.
### 4.
**Microsoft Learn (2023). _Windows Defender Event IDs and Status Codes_.** _Description:_ Reference guide for Defender operational event codes, helping verify that telemetry was forwarding correctly.
