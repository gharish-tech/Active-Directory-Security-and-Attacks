# Sysmon — Active Directory Security Lab

## 1. Objective

The objective of this module is to install Microsoft Sysmon, verify that its service and event log are available, and capture a process-creation event.

This exercise demonstrates how endpoint telemetry helps analysts understand which process started, which account ran it, and which process launched it.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| Domain Controller IP | `192.168.182.10` |
| Operating system | Windows Server 2022 |
| Account used for the test | `HARISH\Administrator` |
| Tool | Microsoft Sysmon 15.22 |

## 3. What Is Sysmon?

System Monitor (Sysmon) is a Windows system service and driver from Microsoft Sysinternals. It records selected system activity into the Windows Event Log.

Sysmon is useful for endpoint investigation because its events can include details such as:

- Process creation and command-line information.
- Parent and child process relationships.
- Executable hashes.
- Network connections, when configured.
- Other selected file, image-load, and DNS activity.

Sysmon provides telemetry. It does not independently determine that every recorded event is malicious, and it does not automatically block an attack.

## 4. Installation

Sysmon was downloaded from the official Microsoft Sysinternals page and extracted to:

`C:\Users\Administrator\Downloads\Sysmon`

The extracted directory contained `Sysmon.exe`, `Sysmon64.exe`, and `Sysmon64a.exe`.

The following command was run from an elevated PowerShell session:

```powershell
.\Sysmon64.exe -accepteula -i
```

### Observed installation result

```text
Sysmon64 installed.
SysmonDrv installed.
Starting SysmonDrv.
SysmonDrv started.
Starting Sysmon64..
Sysmon64 started.
```

This confirms that the installation command reported success and the service and driver started.

## 5. Verify the Service and Event Log

The service was checked with:

```powershell
Get-Service Sysmon*
```

Observed result:

```text
Status   Name       DisplayName
------   ----       -----------
Running  Sysmon64   Sysmon64
```

The event log was checked with:

```powershell
Get-WinEvent -ListLog 'Microsoft-Windows-Sysmon/Operational' |
    Select-Object LogName, IsEnabled, RecordCount
```

Observed result:

```text
LogName                                 IsEnabled RecordCount
-------                                 --------- -----------
Microsoft-Windows-Sysmon/Operational    True      13
```

At the time of this check, the Sysmon Operational log was enabled and contained 13 records.

## 6. Practical: Generate a Process-Creation Event

The following commands were used to launch Notepad and query recent Sysmon Event ID 1 records:

```powershell
Start-Process notepad.exe
Start-Sleep -Seconds 2

Get-WinEvent -FilterHashtable @{
    LogName   = 'Microsoft-Windows-Sysmon/Operational'
    Id        = 1
    StartTime = (Get-Date).AddMinutes(-5)
} -MaxEvents 10 |
    Select-Object TimeCreated, Id, Message |
    Format-List
```

## 7. Observed Event ID 1 Evidence

The supplied event recorded the following information:

| Field | Observed value |
|---|---|
| Event ID | `1` |
| Event description | Process Create |
| Image | `C:\Windows\System32\notepad.exe` |
| Process ID | `2284` |
| Parent process ID | `4616` |
| Parent image | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| User | `HARISH\Administrator` |
| Integrity level | `High` |
| Command line | `"C:\Windows\system32\notepad.exe"` |
| File version | `10.0.20348.1` |
| Hash | SHA-256 was recorded |
| Process GUID | Present |
| Parent process GUID | Present |

The event demonstrates that Sysmon recorded Notepad's process creation, its parent PowerShell process, and the account context.

## 8. Process Tree Interpretation

The observed relationship was:

```text
powershell.exe (PID 4616)
└── notepad.exe (PID 2284)
```

This relationship matches the lab action: PowerShell executed `Start-Process notepad.exe`.

Process IDs can be reused over time. Sysmon's process GUIDs provide additional identifiers for correlating events involving the same process.

## 9. Detection Value

Process-creation telemetry can help investigate:

- Unexpected executables launched by administrative tools.
- Scripts or programs executed from unusual directories.
- Suspicious parent-child process relationships.
- Unfamiliar command-line arguments.
- Processes running under unexpected accounts or integrity levels.

These are investigation leads, not automatic proof of compromise. A legitimate administrator may run PowerShell and other command-line tools during normal operations.

The observed Notepad event was expected because the process was deliberately launched for this test.

## 10. Event IDs and Configuration

Common Sysmon event IDs include:

| Event ID | Activity |
|---:|---|
| 1 | Process creation |
| 3 | Network connection, if enabled by configuration |
| 7 | Image loaded, if configured |
| 11 | File created |
| 22 | DNS query |

Not every event type is necessarily enabled in every Sysmon configuration. Event generation depends on the installed Sysmon version and its configuration.

The successful Event ID 1 capture does not prove that network-connection logging or all other telemetry categories are enabled.

## 11. Security and Operational Considerations

- Download Sysmon from Microsoft's official Sysinternals source.
- Use an appropriate Sysmon configuration for the environment.
- Review log volume, retention, and filtering requirements.
- Protect endpoint logs from unauthorized modification.
- Forward important events to a central logging platform when available.
- Correlate Sysmon records with Windows Security logs and other evidence.

This exercise installed Sysmon on the lab domain controller. A production deployment should follow change-management and monitoring requirements because additional logging can affect storage and operational workflows.

## 12. Limitations

- Only installation status and process-creation telemetry were verified here.
- Network connection logging was not demonstrated.
- The test event is benign and does not demonstrate detection of a real attack.
- No SIEM integration or alert rule was tested.
- A recorded event does not establish malicious intent by itself.

## 13. Conclusion

Sysmon was installed successfully on DC01, its service was running, and the Sysmon Operational event log was enabled. A controlled Notepad launch generated Event ID 1 with process, parent-process, user, and hash information.

The exercise demonstrates how Sysmon adds detailed endpoint telemetry that can support investigation and detection when correlated with other security evidence.

