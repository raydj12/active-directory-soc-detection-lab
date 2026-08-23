# Incident 001 - Multiple Failed Logons

## Incident Summary

Microsoft Sentinel generated a Medium severity incident after multiple failed logon attempts were detected against the same user account on WS01.

The activity initially appeared consistent with password guessing because several authentication failures occurred within a short period of time. I reviewed the authentication timeline and surrounding activity to determine whether the attempts were malicious or legitimate.

## Incident Details

- **Severity:** Medium
- **Affected User:** `RAETECH\mreed`
- **Affected Host:** `WS01.RaeTech.local`
- **Source Address:** `127.0.0.1`
- **Relevant Events:** Windows Security Event IDs 4625 and 4624
- **MITRE ATT&CK:** T1110 - Brute Force
- **Final Classification:** Benign Positive - Suspicious But Expected

## Detection

The Sentinel analytics rule monitored Windows Security Event ID **4625**, which represents a failed logon attempt.

The rule generated an incident after multiple failed authentication attempts occurred against the same account within a short time window.

## Investigation

I reviewed the authentication events surrounding the alert and found several Event ID 4625 failures followed by successful Event ID 4624 logons.

The successful authentication activity was associated with `RAETECH\mreed` on WS01. The source address was `127.0.0.1`, and the successful events included cached interactive and workstation unlock activity.

I also reviewed process creation activity around the same time window. I did not find any obvious suspicious process execution associated with the authentication activity.

The evidence was consistent with legitimate local logon attempts rather than a remote brute-force attack.

## Analyst Assessment

The detection itself worked as intended because repeated failed logons can indicate password guessing or brute-force activity.

After reviewing the user, host, source, authentication timeline, and surrounding process activity, I determined that the activity was generated during an authorized lab test.

The incident was classified as:

**Benign Positive - Suspicious But Expected**

## Response Recommendation

No containment or remediation was required in the lab.

In a production environment, I would verify the activity with the affected user and review additional authentication and endpoint telemetry before closing the incident.

If the attempts could not be explained, additional actions could include reviewing other affected accounts, checking for remote authentication attempts, resetting credentials, or escalating the incident.

## Lessons Learned

This investigation reinforced that an alert does not automatically confirm malicious activity.

Repeated authentication failures can look similar to brute-force behavior, but additional context such as the source, logon type, successful authentication events, and surrounding endpoint activity is necessary before making a final determination.
