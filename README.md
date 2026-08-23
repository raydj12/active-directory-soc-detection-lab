# Active Directory SOC Detection Lab

## Project Overview

I built this lab to practice the detection and investigation workflow used by SOC analysts in a Windows Active Directory environment.

The lab includes a Windows Server domain controller, a domain-joined Windows workstation, Microsoft Sentinel, Windows security logging, Sysmon, PowerShell logging, and a Kali Linux system used to generate controlled attack activity.

Throughout the project, I used KQL to analyze authentication and process events, created custom Microsoft Sentinel detection rules, investigated the resulting incidents, and documented how I determined whether activity was suspicious or expected.

## Objective

The objective of this lab was to build a small enterprise-style Windows environment and practice the day-to-day workflow of a SOC analyst.

I wanted to understand how activity on a Windows endpoint becomes security telemetry, how that telemetry reaches Microsoft Sentinel, and how an analyst can use KQL and detection rules to identify and investigate suspicious behavior.

The main goals of the project were to:

- Build and manage an Active Directory domain
- Configure a domain-joined Windows workstation
- Enable useful Windows, Sysmon, and PowerShell logging
- Send Windows security events to Microsoft Sentinel
- Use KQL to investigate authentication and process activity
- Create custom Sentinel analytics rules
- Generate controlled attack activity using Kali Linux
- Investigate Sentinel incidents and determine whether the activity was malicious or expected
- Document findings using SOC-style incident reports

## Lab Architecture

The lab was built in VirtualBox and designed to represent a small enterprise environment with centralized identity, endpoint logging, attack simulation, and SIEM monitoring.

The main systems used in the lab were:

- **DC01** - Windows Server domain controller running Active Directory Domain Services and DNS
- **WS01** - Windows 11 domain-joined workstation used as the monitored endpoint
- **Kali Linux** - Used as a simulated attacker for controlled network reconnaissance and remote authentication testing
- **Microsoft Sentinel** - Used as the SIEM for log collection, KQL analysis, detection rules, and incident investigation
- **Azure Arc** - Connected and represented WS01 as an Azure-managed machine
- **Azure Monitor Agent** - Collected selected Windows Security events from WS01 and sent them to the Log Analytics workspace through a Data Collection Rule

The general data flow of the lab was:

```text
Kali Linux
Simulated Attacker
      |
      v
    WS01 ---------------- DC01
Windows Endpoint      Domain Controller
      |
      | Windows Security Events
      v
Azure Monitor Agent
      |
      v
Log Analytics Workspace
      |
      v
Microsoft Sentinel
      |
      +--> KQL Queries
      +--> Detection Rules
      +--> Alerts / Incidents
      |
      v
SOC Investigation
```

## Technologies Used

| Technology | Purpose in the Lab |
|---|---|
| **Oracle VirtualBox** | Hosted the virtual machines used in the lab |
| **Windows Server** | Hosted the RaeTech.local Active Directory domain and DNS services |
| **Windows 11 Pro** | Served as the domain-joined workstation and monitored endpoint |
| **Active Directory Domain Services** | Managed domain users, groups, computers, and Group Policy |
| **Group Policy** | Applied security and auditing settings to WS01 |
| **Sysmon** | Provided additional endpoint process telemetry |
| **Windows Security Auditing** | Recorded authentication and process creation events |
| **PowerShell Script Block Logging** | Provided visibility into PowerShell activity |
| **Microsoft Defender** | Provided built-in endpoint protection on WS01 |
| **Azure Arc** | Connected the lab workstation to Azure as a managed machine |
| **Azure Monitor Agent** | Collected Windows Security events from WS01 |
| **Data Collection Rules** | Controlled which Windows events were sent to Azure |
| **Log Analytics Workspace** | Stored and allowed querying of collected security telemetry |
| **Microsoft Sentinel** | Provided SIEM detection, alerting, and incident investigation |
| **Kusto Query Language (KQL)** | Used to search and analyze security events |
| **Kali Linux** | Generated controlled reconnaissance and remote authentication activity |
| **Nmap** | Performed network and service reconnaissance against the lab endpoint |
| **SMB / smbclient** | Generated and tested remote Windows authentication activity |
