# Splunk SIEM Lab with Sysmon and Windows Event Monitoring

## Overview

This project is a hands-on SIEM lab built using Splunk, Sysmon, and a Windows virtual machine. I created this lab to better understand how endpoint activity is collected, forwarded, and analyzed in a centralized logging environment.

The idea behind the project was to simulate a basic real-world security monitoring workflow. I used a Windows VM as the endpoint, installed Sysmon to capture detailed system activity, and configured Splunk Universal Forwarder to send logs to Splunk Enterprise running on my host machine. Once the data pipeline was working, I used Splunk to search for process creation events, PowerShell activity, and failed login attempts.

This project helped me move beyond theory and actually build a working environment where I could generate events, troubleshoot log forwarding problems, and validate security detections in Splunk.

---

## Why I Built This Project

I built this project to get practical experience with tools and concepts that are commonly used in security monitoring and SOC environments. I wanted to understand not just what Splunk and Sysmon do, but how they work together in an actual logging pipeline.

A lot of learning resources explain SIEM tools at a high level, but this project gave me a chance to set everything up myself, fix issues when things did not work, and see how endpoint activity appears inside Splunk. That made the learning much more real and much more useful.

---

## What This Project Demonstrates

This lab demonstrates how to:

- build a small Windows security monitoring environment
- collect detailed endpoint telemetry using Sysmon
- forward logs from a monitored endpoint into Splunk
- search and analyze Windows Security and Sysmon logs
- create simple detection searches for suspicious or important activity
- validate detections by generating test events in the endpoint
- document a security monitoring lab in a professional GitHub format

---

## Tools Used

- **VirtualBox** – used to create and run the Windows virtual machine
- **Windows VM** – used as the monitored endpoint
- **Sysmon** – used to capture detailed endpoint events
- **Splunk Universal Forwarder** – used to forward logs from the Windows VM
- **Splunk Enterprise** – used as the SIEM platform on the host machine
- **Windows Event Viewer** – used to validate logs locally inside the VM

---

## Lab Architecture

The setup for this lab is shown below in simple form:

**Windows VM → Sysmon / Windows Event Logs → Splunk Universal Forwarder → Splunk Enterprise**

### Component Roles

**Windows VM**  
This machine acts as the endpoint being monitored. It is where user activity and authentication activity take place.

**Sysmon**  
Sysmon generates detailed Windows events that are very useful for security monitoring, especially process creation events and execution activity.

**Windows Security Logs**  
These logs capture authentication and account-related events, including failed logins.

**Splunk Universal Forwarder**  
The forwarder reads the selected Windows logs from the VM and sends them to Splunk Enterprise on the host machine.

**Splunk Enterprise**  
Splunk receives the logs, indexes them, and makes them searchable for analysis, alerting, and visualization.

---

## Project Setup Summary

## 1. Created the Windows Virtual Machine

I first created a Windows virtual machine in VirtualBox. This VM served as the monitored endpoint for the lab. It allowed me to safely generate activity and test different detections without affecting my main system.

## 2. Installed Sysmon

After the VM was ready, I installed Sysmon inside the Windows machine. Sysmon was important because default Windows logs alone do not always provide enough detail for monitoring process activity.

I confirmed that Sysmon was working by checking this log path in Event Viewer:

`Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational`

Once I saw Sysmon events there, I knew the endpoint was generating the telemetry I needed.

## 3. Installed Splunk Enterprise on the Host Machine

I installed Splunk Enterprise on my main machine and used it as the central SIEM server. This allowed me to collect and search logs from the VM in one place.

I also configured Splunk to receive forwarded data on port `9997`.

## 4. Installed Splunk Universal Forwarder on the Windows VM

Inside the Windows VM, I installed Splunk Universal Forwarder. The purpose of the forwarder was to send selected logs from the endpoint to Splunk Enterprise running on the host.

I configured it to forward data to the host machine IP and Splunk receiving port.

## 5. Configured Log Collection

To tell the forwarder which logs to collect, I created an `inputs.conf` file and added the Windows Security and Sysmon channels.

The configuration used in this project is shown below:

```ini
[WinEventLog://Security]
disabled = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
```

## 6. Troubleshot the Sysmon Forwarding Issue

At first, only Windows Security logs appeared in Splunk, while Sysmon logs did not. I investigated the issue by checking Splunk configuration loading and forwarder logs.

The problem turned out to be a permissions issue. The SplunkForwarder service was running under the wrong account and did not have the required access to subscribe to the Sysmon event channel. After changing the service to run as **Local System**, Sysmon logs were successfully forwarded.

This was one of the most valuable parts of the project because it showed me how to troubleshoot a real log collection problem instead of just following setup instructions.

---

## Detection Searches Created

Once the lab was working, I created and validated several useful Splunk searches.

## 1. Sysmon Process Creation

This search identifies Sysmon EventCode 1 process creation events from the monitored endpoint.

```spl
index=* host=WIN-SOC-LAB source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
```

This search helped confirm that process execution activity from the VM was being captured in Splunk.

---

## 2. PowerShell Execution

This search focuses on PowerShell-related process creation activity using Sysmon logs.

```spl
index=* host=WIN-SOC-LAB source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 (Image="*\\powershell.exe" OR Image="*\\pwsh.exe")
```

I validated this search by opening PowerShell inside the Windows VM and executing a simple command. The resulting event showed the executable path, command line, parent process, and user context.

---

## 3. Failed Login Attempts

This search identifies failed Windows login attempts using Security EventCode 4625.

```spl
index=* host=WIN-SOC-LAB source="WinEventLog:Security" EventCode=4625
```

I tested this by intentionally entering the wrong password several times on the Windows VM login screen. Splunk successfully captured and returned the failed login events.

---

## 4. Successful Login Events

This search can be used to review successful authentication activity using Security EventCode 4624.

```spl
index=* host=WIN-SOC-LAB source="WinEventLog:Security" EventCode=4624
```

---

## Dashboard Created

After validating the searches, I created a basic Splunk dashboard called:

**Windows SIEM Lab Dashboard**

The dashboard includes panels for:

- Sysmon Process Creation
- PowerShell Execution
- Failed Logins

The purpose of the dashboard was to give a simple visual view of the most important monitored activity in the lab and make the project feel more like a real SIEM workflow instead of just a set of individual searches.

---

## Screenshots

This repository includes screenshots of the key parts of the project, including:

- Sysmon process creation events
- PowerShell execution events
- failed login events
- dashboard panels

These screenshots are included to show that the setup was fully tested and working.

---

## Project Structure

```text
splunk-siem-sysmon-lab/
├── README.md
├── configs/
│   └── inputs.conf
├── searches/
│   ├── sysmon_process_creation.txt
│   ├── powershell_execution.txt
│   ├── failed_logins.txt
│   └── successful_logins.txt
├── screenshots/
│   ├── sysmon_process_creation.png
│   ├── powershell_execution.png
│   ├── failed_logins.png
│   └── dashboard.png
└── docs/
    └── project_notes.md
```

---

## What I Learned

This project taught me several practical lessons.

First, I learned how endpoint logging works in a real pipeline. Logs do not just appear in a SIEM automatically. You need a monitored system, the right telemetry source, a forwarding agent, correct configuration, correct permissions, and a receiving SIEM platform.

Second, I learned the difference between Windows Security logs and Sysmon logs. Security logs are helpful for authentication and account activity, while Sysmon provides much more detailed visibility into execution behavior and endpoint events.

Third, I learned how Splunk Universal Forwarder works in a basic lab environment. I also learned how configuration files like `inputs.conf` control data collection.

Most importantly, I learned how to troubleshoot when data collection fails. The Sysmon forwarding issue forced me to investigate configuration loading, service settings, and log subscription permissions. That troubleshooting experience was one of the most valuable outcomes of the project.

---

## Why This Project Matters

This project is useful because it reflects the type of workflow used in real security operations and monitoring roles. It demonstrates practical experience with:

- Windows logging
- Sysmon
- Splunk
- log forwarding
- endpoint visibility
- event analysis
- security detection validation
- troubleshooting data ingestion issues

This makes the project relevant for roles such as:

- SOC Analyst
- Cybersecurity Analyst
- Security Monitoring Analyst
- Blue Team Analyst
- SIEM Analyst
- Detection Engineer

---

## Future Improvements

There are several ways this project could be extended in the future, including:

- adding more Sysmon event types and tuning the configuration
- creating Splunk alerts instead of only saved searches
- building more advanced dashboards
- simulating additional endpoint activity
- adding network-based detections
- testing suspicious PowerShell command patterns
- mapping searches to MITRE ATT&CK techniques

---

## Conclusion

This project gave me a practical understanding of how a SIEM pipeline works from end to end. Instead of only studying Splunk and Windows logging in theory, I built a working lab where I could generate endpoint activity, forward logs, troubleshoot ingestion issues, and validate detections.

It was a strong hands-on exercise in security monitoring and helped me better understand how logs are collected, searched, and used for basic threat detection in a Windows environment.
