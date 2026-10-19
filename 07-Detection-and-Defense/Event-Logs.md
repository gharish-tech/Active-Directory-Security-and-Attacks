# Windows Events and Logs — Active Directory Security Lab

## 1. Objective

This module explains how to collect and interpret Windows Security events and Sysmon telemetry in the Active Directory lab.

The goal is to understand the purpose of each log source, identify relevant event IDs, and document evidence without assuming that unusual activity is automatically malicious.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| Domain Controller IP | `192.168.182.10` |
| Windows Client | `CLIENT01` |
| Security log | Windows Security |
| Endpoint telemetry | Sysmon Operational |

## 3. Windows Security Log

The Windows Security log records security-related activity according to the configured audit policies. Depending on the configuration, it can record authentication, account management, directory-service activity, and access to resources.

### Query authentication-related events

```powershell
Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = 4624, 4625, 4768, 4769, 4771
    StartTime = (Get-Date).AddHours(-24)
} -MaxEvents 20 |
Select-Object TimeCreated, Id, ProviderName, Message |
Format-List
```

This read-only query retrieves up to 20 matching events from the previous 24 hours.

### Relevant event IDs

| Event ID | Description | Useful fields |
|---:|---|---|
| 4624 | Successful logon | Account, logon type, source address, authentication package |
| 4625 | Failed logon | Account, source address, failure reason |
| 4768 | Kerberos TGT request | Account, client address, result code |
| 4769 | Kerberos service-ticket request | Account, service name, client address |
| 4771 | Kerberos pre-authentication failure | Account, client address, failure code |
| 4720 | User account created | New account and subject |
| 4728 | Member added to a global security group | Group, added member, subject |
| 4732 | Member added to a local security group | Group, added member, subject |
| 5136 | Directory object modified | Object, attribute and subject, when auditing is configured |

Event availability depends on audit policy, operating-system configuration, and retention.

## 4. Sysmon Operational Log

Sysmon provides additional endpoint telemetry through its own event log. It complements Windows Security auditing rather than replacing it.

### Query process-creation events

```powershell
Get-WinEvent -FilterHashtable @{
    LogName   = 'Microsoft-Windows-Sysmon/Operational'
    Id        = 1
    StartTime = (Get-Date).AddHours(-24)
} -MaxEvents 10 |
Select-Object TimeCreated, Id, Message |
Format-List
```

### Relevant Sysmon event IDs

| Event ID | Description |
|---:|---|
| 1 | Process creation |
| 3 | Network connection, when enabled |
| 7 | Image loaded, when configured |
| 11 | File created |
| 22 | DNS query |

The events that are recorded depend on the Sysmon configuration. A missing event does not necessarily mean that the activity did not occur.

## 5. Compare the Log Sources

| Investigation question | Useful source |
|---|---|
| Was an account successfully logged on? | Windows Security Event 4624 |
| Were there failed logon attempts? | Windows Security Event 4625 |
| Was a Kerberos TGT requested? | Windows Security Event 4768 |
| Was a Kerberos service ticket requested? | Windows Security Event 4769 |
| Which process was launched? | Sysmon Event 1 |
| Which process launched another process? | Sysmon Event 1 parent-process fields |
| Was a network connection recorded? | Sysmon Event 3, if enabled |
| Was a directory object modified? | Security Event 5136, if auditing is configured |

Correlate records using timestamps, account names, hostnames, IP addresses, process IDs, and other available identifiers. Account names and timestamps alone may not be sufficient to prove that two events belong to the same activity.

## 6. Practical Investigation Workflow

1. Identify the event and the system where it was recorded.
2. Record the timestamp and time zone.
3. Identify the account, source address, and target service where available.
4. Inspect the event's detailed fields.
5. Correlate relevant Windows Security and Sysmon records.
6. Compare the activity with expected lab operations.
7. Document the observed evidence, possible explanations, and unresolved questions.
8. Classify the result as expected, suspicious, or confirmed malicious only when the evidence supports that conclusion.

## 7. Evidence Handling

For each important event, record:

- Event source and event ID.
- Original timestamp and time zone.
- Account and host involved.
- Relevant source and destination details.
- Process information, where available.
- Why the event is relevant.
- Supporting and contradictory evidence.
- Investigation conclusion and confidence.

When retaining evidence, preserve the original event details and avoid changing timestamps or omitting fields that could alter interpretation.

## 8. Lab Observations

The lab's supplied Windows Security output included successful logon events involving the `DC01$` computer account and the `SYSTEM` context.

Sysmon was subsequently installed on DC01. The service was verified as running, and the Sysmon Operational log was enabled.

A controlled Notepad launch generated Sysmon Event ID 1. The event recorded `notepad.exe`, its parent `powershell.exe`, the account `HARISH\Administrator`, process identifiers, and a SHA-256 hash.

The Notepad launch was expected because it was deliberately initiated during the lab. It demonstrates telemetry collection, not attack detection.

## 9. Limitations

- Only the event records actually collected are considered observed evidence.
- Audit policies affect which Windows Security events are available.
- Sysmon event collection depends on its configuration.
- The absence of a record is not proof that an action did not occur.
- An event by itself rarely establishes malicious intent.
- No SIEM alert or centralized log pipeline was demonstrated in this module.

## 10. Conclusion

Windows Security logs and Sysmon provide complementary evidence for Active Directory security investigations. Security events help investigate authentication and account activity, while Sysmon adds endpoint details such as process creation and parent-child relationships.

Reliable detection requires context, correlation, and careful documentation. The evidence collected in this lab provides a foundation for further monitoring and detection exercises.

