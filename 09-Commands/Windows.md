# Windows Cheatsheet

## 1. System Information

| Command | Purpose |
|---|---|
| `hostname` | Display computer name |
| `whoami` | Display current user |
| `whoami /all` | Display user, groups, and privileges |
| `systeminfo` | Display detailed system information |
| `ver` | Display Windows version information |
| `echo %USERNAME%` | Display current username |
| `echo %COMPUTERNAME%` | Display computer name |

## 2. Files and Directories

| Command | Purpose |
|---|---|
| `dir /a` | List files, including hidden items |
| `cd` | Display or change the current directory |
| `mkdir Lab` | Create a directory |
| `type file.txt` | Display text-file contents |
| `copy source destination` | Copy a file |
| `move source destination` | Move a file |
| `where python` | Locate an executable using PATH |
| `tree /f` | Display directory structure and files |

## 3. Networking

| Command | Purpose |
|---|---|
| `ipconfig /all` | Display interfaces, addresses, gateways, and DNS settings |
| `ipconfig /displaydns` | Display the DNS resolver cache |
| `ipconfig /flushdns` | Clear the DNS resolver cache |
| `ping 192.168.182.10` | Test ICMP reachability |
| `tracert 192.168.182.10` | Trace the network path |
| `nslookup harish.local` | Query DNS |
| `route print` | Display the routing table |
| `arp -a` | Display the ARP cache |
| `netstat -ano` | Display connections, listening ports, and PIDs |
| `Test-NetConnection 192.168.182.10 -Port 445` | Test SMB TCP connectivity from PowerShell |

## 4. Users and Groups

| Command | Purpose |
|---|---|
| `whoami /groups` | Display current user's group memberships |
| `net user` | List local users |
| `net user USERNAME` | Display information about a local or domain user, depending on context |
| `net localgroup` | List local groups |
| `net localgroup Administrators` | Inspect local Administrators group membership |
| `net group /domain` | List domain groups when run with suitable domain connectivity and permissions |

## 5. Services and Processes

| Command | Purpose |
|---|---|
| `tasklist` | List running processes |
| `tasklist /svc` | Associate processes with services |
| `sc query` | Query service information |
| `sc query Sysmon64` | Inspect the Sysmon service |
| `services.msc` | Open the Services management console |
| `taskmgr` | Open Task Manager |
| `eventvwr.msc` | Open Event Viewer |

## 6. SMB and Shares

| Command | Purpose |
|---|---|
| `net view \\DC01` | Attempt to list visible shares on DC01 |
| `net use` | Display current network connections |
| `net share` | List shares on the local computer |
| `Get-SmbShare` | List local SMB shares from PowerShell |
| `Get-SmbConnection` | Inspect SMB client connections from PowerShell |

## 7. Troubleshooting Sequence

For basic network troubleshooting, investigate in this order:

1. `ipconfig /all` — check the IP address, subnet mask, gateway, and DNS servers.
2. `ping 192.168.182.10` — test reachability to the domain controller.
3. `nslookup DC01.harish.local` — check name resolution.
4. `Test-NetConnection 192.168.182.10 -Port 445` — test SMB connectivity.
5. Review the relevant Windows logs and service status.

A failed ping does not necessarily mean the host is offline; ICMP may be blocked. Likewise, an open TCP port does not prove that authentication will succeed.

## Important Notes

- Run CMD commands in Command Prompt unless otherwise specified.
- Commands may behave differently depending on whether the computer is domain-joined.
- Administrative tasks require appropriate permissions.
- Do not publish sensitive command output, credentials, or private configuration in a public repository.
