# Incident 004 - Network Service Scanning

## Incident Summary

Microsoft Sentinel generated a Medium severity incident after detecting network service scanning activity against WS01.

The activity came from the Kali Linux lab machine and involved one source IP contacting many different destination ports on WS01 within a short period of time. I reviewed the Windows network telemetry and Sentinel activity to determine whether the scan was malicious or expected.

## Incident Details

- **Severity:** Medium
- **Source Host:** Kali Linux
- **Source IP:** `192.168.20.103`
- **Destination Host:** `WS01`
- **Destination IP:** `192.168.20.105`
- **Relevant Event:** Windows Security Event ID 5152
- **Protocol:** TCP
- **MITRE ATT&CK:** T1046 - Network Service Scanning
- **Final Classification:** Benign Positive - Suspicious But Expected

## Detection

I used Nmap from the Kali Linux VM to perform a basic network service scan against WS01.

Windows Filtering Platform blocked many of the connection attempts and generated Security Event ID **5152**.

The Sentinel analytics rule looked for one source IP contacting many different destination ports within a short time window. The large number of packet-drop events from the same source matched the detection conditions and generated an incident.

## Investigation

I first reviewed Event ID 5152 locally on WS01 and confirmed that multiple blocked packets were being generated during the scan.

The events showed:

- Source IP `192.168.20.103`
- Destination IP `192.168.20.105`
- Multiple different destination ports
- TCP traffic
- Multiple events occurring within only a few seconds

I then verified that Event ID 5152 was being ingested into Microsoft Sentinel through the `SecurityEvent` table.

During the investigation, I confirmed that the source IP belonged to the Kali Linux machine in my lab and that the timing of the activity matched the Nmap scan I had intentionally performed.

## Analyst Assessment

The detection correctly identified behavior consistent with network service scanning.

A single blocked connection would not necessarily indicate reconnaissance. The suspicious pattern was created by the same source IP contacting many different destination ports within a short period of time.

Because the source and timing matched an authorized lab test, I determined that the activity was expected.

The incident was classified as:

**Benign Positive - Suspicious But Expected**

## Response Recommendation

No containment or remediation was required because the scan was authorized lab activity.

In a production environment, I would investigate the source system, determine whether the scan was approved, review other activity from the source IP, and check whether additional hosts were targeted.

If the scan could not be explained, I would consider isolating the source system, reviewing endpoint telemetry, and escalating the incident for further investigation.

## Lessons Learned

This incident reinforced that a detection can be accurate even when the final activity is benign.

The rule correctly identified the scanning pattern, but the investigation provided the context needed to determine whether the activity was authorized.

I also learned that endpoint telemetry must actually be collected by the SIEM before it can be used for detection. Event ID 5152 was initially available only on WS01, so I updated the Azure Data Collection Rule and verified that the events were successfully reaching Sentinel.
