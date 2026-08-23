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
