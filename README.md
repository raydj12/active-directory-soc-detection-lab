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

## Active Directory Environment

I started the lab by building a small Windows domain to represent the identity environment of a business.

The domain controller, **DC01**, was configured with Active Directory Domain Services and DNS for the `RaeTech.local` domain. I created organizational units, users, and security groups to give the environment a basic enterprise structure.

A Windows 11 Pro workstation named **WS01** was then joined to the domain. I verified that a domain user could successfully sign in to the workstation using Active Directory credentials.

### Domain Configuration

| Component | Configuration |
|---|---|
| **Domain** | RaeTech.local |
| **Domain Controller** | DC01 |
| **Domain Services** | Active Directory Domain Services, DNS |
| **Domain Workstation** | WS01 |
| **Workstation OS** | Windows 11 Pro |
| **Example Domain User** | RAETECH\mreed |
| **Identity Structure** | Organizational Units, users, and security groups |

### Active Directory OU Structure

I organized the `RaeTech.local` domain into Organizational Units to separate users and systems by function. This gave the lab a more realistic enterprise structure and provided a foundation for applying policies and managing domain objects.

![Active Directory OU Structure](active-directory/04-active-directory-ou-structure.png)

*Active Directory Users and Computers showing the organizational structure created for the RaeTech.local domain.*

### Domain User Login Verification

After joining WS01 to the `RaeTech.local` domain, I verified that a domain user could successfully authenticate to the workstation using Active Directory credentials.

![Domain User Login Verification](active-directory/06-domain-user-login-verification.png)

*Successful domain authentication on WS01 using a RaeTech.local user account.*

## Endpoint Security & Logging

After building the domain environment, I configured WS01 to generate security telemetry that could later be analyzed in Microsoft Sentinel.

I enabled process creation auditing, PowerShell Script Block Logging, and Sysmon to provide visibility into authentication activity, commands, and processes running on the workstation.

### Process Creation Auditing

I enabled Windows process creation auditing through Group Policy. This allowed Windows Security Event ID **4688** to record newly created processes and their command-line information.

This became important later when investigating commands such as `net user`, `whoami`, `hostname`, and `ipconfig`.

![Process Creation Auditing GPO](screenshots/07-process-creation-auditing-gpo-verification.png)

*Group Policy configuration used to enable process creation auditing on WS01.*

### PowerShell Script Block Logging

PowerShell Script Block Logging was enabled to provide additional visibility into PowerShell commands executed on the workstation.

I verified the configuration by confirming PowerShell Event ID **4104** was generated.

![PowerShell Script Block Logging](screenshots/08-powershell-scriptblock-4104-verification.png)

*PowerShell Event ID 4104 confirming Script Block Logging was working.*

### Sysmon

I installed Sysmon on WS01 and applied a custom configuration to increase endpoint visibility beyond the standard Windows Security logs.

Sysmon provided detailed process telemetry that could be used to review process execution and parent-child relationships during an investigation.

![Sysmon Process Creation](screenshots/10-sysmon-process-creation-event.png)

*Sysmon process creation telemetry generated on WS01.*

## Microsoft Sentinel Integration

After configuring security logging on WS01, I connected the workstation to Azure so the Windows Security events could be analyzed in Microsoft Sentinel.

WS01 was onboarded to **Azure Arc** and monitored using the **Azure Monitor Agent**. I created a Data Collection Rule to collect selected Windows Security events and send them to the `RaeTech-Sentinel-Workspace` Log Analytics workspace.

The main Windows Security events collected for this lab included:

- **4624** - Successful logon
- **4625** - Failed logon
- **4688** - Process creation

The telemetry path used in the lab was:

```text
WS01
  |
  | Windows Security Events
  v
Azure Monitor Agent
  |
  | Data Collection Rule
  v
Log Analytics Workspace
  |
  v
Microsoft Sentinel
  |
  +--> KQL Queries
  +--> Analytics Rules
  +--> Alerts / Incidents
```

## Microsoft Sentinel Integration

After configuring security logging on WS01, I onboarded the workstation to Azure Arc and installed the Azure Monitor Agent.

I created a Data Collection Rule to collect selected Windows Security events and send them to the `RaeTech-Sentinel-Workspace` Log Analytics workspace, where Microsoft Sentinel was enabled for detection and investigation.

The primary Windows Security events collected for the lab included:

- **4624** - Successful logon
- **4625** - Failed logon
- **4688** - Process creation

### Security Event Ingestion Verification

I used KQL to verify that Windows Security events from WS01 were successfully reaching the Log Analytics workspace.

![Sentinel Failed Logon KQL](sentinel/14-sentinel-kql-failed-logon-4625.png)

*Windows Event ID 4625 from WS01 successfully ingested and queried in Microsoft Sentinel.*

## Detection Engineering & SOC Investigation

After confirming that Windows telemetry was reaching Microsoft Sentinel, I used KQL to create detection logic for activity that could require SOC investigation.

For each scenario, I generated the activity, verified the Windows events, searched the telemetry with KQL, created a Sentinel analytics rule, and investigated the resulting incident before deciding how it should be classified.

### Detection 1 - Multiple Failed Logons

The first detection monitored repeated failed authentication attempts on WS01 using Windows Security Event ID **4625**.

Repeated login failures in a short period of time can be associated with password guessing or brute-force activity, but failed logons can also happen for legitimate reasons. Because of that, I treated the alert as something that required investigation rather than assuming it was an attack.

The detection logic counted failed authentication attempts against the same account and host:

```kusto
SecurityEvent
| where EventID == 4625
| where Computer startswith "WS01"
| summarize FailedAttempts=count(),
            FirstAttempt=min(TimeGenerated),
            LastAttempt=max(TimeGenerated)
    by Account, Computer
| where FailedAttempts >= 3
```

Microsoft Sentinel generated a **Medium severity** incident after the detection threshold was reached.

![Failed Logon Incident Created](detections/15-sentinel-failed-logon-incident-created.png)

*Microsoft Sentinel incident created after multiple failed authentication events were detected on WS01.*

#### Investigation

I reviewed the authentication timeline and found multiple failed logons followed by successful authentication events for `RAETECH\mreed`.

The successful activity originated from the local workstation and was consistent with cached interactive login and workstation unlock activity. I also reviewed process activity around the same time and did not find obvious suspicious process execution during the reviewed window.

Based on the available evidence and the fact that the activity was intentionally generated as part of the lab, I classified the incident as:

**Benign Positive - Suspicious But Expected**

![Failed Logon Incident Closed](detections/16-sentinel-incident-closed-benign-positive.png)

*The incident was closed after the authentication activity was determined to be authorized lab testing.*

**MITRE ATT&CK:** T1110 - Brute Force
