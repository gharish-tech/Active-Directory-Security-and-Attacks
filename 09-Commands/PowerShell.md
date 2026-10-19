# PowerShell Cheatsheet

## 1. Basic Commands

| Command | Purpose |
|---|---|
| `Get-Help Get-Process` | Display command help |
| `Get-Command *network*` | Search commands by name |
| `Get-Location` | Show current location |
| `Set-Location C:\Lab` | Change directory |
| `Get-ChildItem -Force` | List files, including hidden items |
| `Get-History` | Display current session command history |
| `Clear-Host` | Clear the console display |

## 2. Files and Text

| Command | Purpose |
|---|---|
| `Get-Content file.txt` | Read a file |
| `Get-Content file.txt -Tail 20` | Read the last 20 lines |
| `Select-String -Path file.txt -Pattern "error"` | Search file contents |
| `Copy-Item source destination` | Copy an item |
| `Move-Item source destination` | Move or rename an item |
| `Test-Path C:\Lab` | Check whether a path exists |
| `New-Item -ItemType Directory -Path C:\Lab\Test` | Create a directory |

## 3. Processes and Services

| Command | Purpose |
|---|---|
| `Get-Process` | List processes |
| `Get-Process notepad` | Find Notepad processes |
| `Get-Service` | List services |
| `Get-Service Sysmon64` | Inspect the Sysmon service |
| `Get-CimInstance Win32_OperatingSystem` | Inspect operating-system information |
| `Get-ScheduledTask` | List scheduled tasks |

## 4. Networking

| Command | Purpose |
|---|---|
| `Get-NetIPConfiguration` | Inspect IP configuration |
| `Get-NetIPAddress` | List IP addresses |
| `Get-NetRoute` | Inspect routing entries |
| `Get-DnsClientServerAddress` | Inspect DNS server settings |
| `Resolve-DnsName DC01.harish.local` | Resolve the lab domain controller name |
| `Test-Connection 192.168.182.10` | Test ICMP reachability |
| `Test-NetConnection 192.168.182.10 -Port 445` | Test TCP connectivity to SMB |
| `Get-NetTCPConnection` | Inspect TCP connections |

## 5. Active Directory

These commands require the Active Directory PowerShell module and appropriate permissions.

| Command | Purpose |
|---|---|
| `Get-ADDomain` | Inspect domain information |
| `Get-ADForest` | Inspect forest information |
| `Get-ADUser -Filter *` | List users the caller can enumerate |
| `Get-ADGroup -Filter *` | List groups the caller can enumerate |
| `Get-ADComputer -Filter *` | List computer accounts |
| `Get-ADGroupMember "Domain Admins"` | Inspect direct group membership |
| `Get-ADOrganizationalUnit -Filter *` | List organizational units |
| `Get-GPO -All` | List Group Policy Objects; requires the GroupPolicy module |

## 6. Windows Event Logs

| Command | Purpose |
|---|---|
| `Get-WinEvent -ListLog *` | List event logs |
| `Get-WinEvent -LogName System -MaxEvents 20` | Read recent System events |
| `Get-WinEvent -LogName Security -MaxEvents 20` | Read recent Security events when authorized |
| `Get-Service Sysmon64` | Check Sysmon service status |

Example: find recent process-creation events from Sysmon:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
    StartTime = (Get-Date).AddHours(-1)
} -MaxEvents 20
```

## 7. Output Filtering

| Command | Purpose |
|---|---|
| `Get-Process \| Where-Object CPU -gt 10` | Filter processes by CPU value |
| `Get-Service \| Select-Object Name, Status` | Select specific properties |
| `Get-Process \| Sort-Object CPU -Descending` | Sort processes |
| `Get-Process \| Export-Csv processes.csv -NoTypeInformation` | Export process data to CSV |

## Important Notes

- PowerShell commands return structured objects, not just text.
- `Where-Object` filters objects; `Select-Object` chooses properties.
- Some commands require an elevated shell, a particular module, or appropriate permissions.
- Review a command's effects before running it against directory objects or system configuration.
