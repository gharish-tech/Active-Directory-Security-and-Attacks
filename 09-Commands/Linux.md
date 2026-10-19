# Linux Cheatsheet

## 1. Navigation

| Command | Purpose |
|---|---|
| `pwd` | Display current directory |
| `ls -la` | List files, including hidden files |
| `cd /path` | Change directory |
| `cd ..` | Move to parent directory |
| `find /path -name "file"` | Find files by name |
| `which python3` | Locate a command |
| `man ls` | Open a command manual |

## 2. Files and Directories

| Command | Purpose |
|---|---|
| `touch file.txt` | Create an empty file |
| `mkdir project` | Create a directory |
| `cp source destination` | Copy files |
| `mv source destination` | Move or rename files |
| `cat file.txt` | Display file contents |
| `less file.txt` | Read a file page by page |
| `head -n 20 file.txt` | Display the first 20 lines |
| `tail -n 20 file.txt` | Display the last 20 lines |
| `grep -n "error" file.txt` | Search for matching lines |

## 3. Permissions and Users

| Command | Purpose |
|---|---|
| `id` | Show current user and group IDs |
| `whoami` | Show current username |
| `groups` | List current user's groups |
| `ls -l` | Inspect file permissions |
| `chmod 640 file.txt` | Set owner read/write, group read, others no access |
| `chown user:group file.txt` | Change file ownership; normally requires appropriate privileges |
| `sudo command` | Run a command with elevated privileges when authorized |

## 4. Processes and Services

| Command | Purpose |
|---|---|
| `ps aux` | List processes |
| `top` | Monitor running processes |
| `pgrep process` | Find process IDs by name |
| `systemctl status ssh` | Inspect service status |
| `journalctl -u ssh` | View service logs |
| `kill PID` | Send the default termination signal to a process |

## 5. Networking

| Command | Purpose |
|---|---|
| `ip addr` | Display network interfaces and addresses |
| `ip route` | Display routing table |
| `ip neigh` | Display neighbor cache |
| `ping IP` | Test basic IP reachability |
| `ss -tulpn` | Inspect listening sockets; process details may require privileges |
| `getent hosts example.com` | Resolve a hostname using system name-service configuration |
| `dig example.com` | Query DNS |
| `curl -I https://example.com` | Request HTTP response headers |
| `traceroute example.com` | Trace a network path when supported |

## 6. Storage and System Information

| Command | Purpose |
|---|---|
| `df -h` | Show filesystem space |
| `du -sh directory` | Summarize directory size |
| `free -h` | Display memory usage |
| `uptime` | Show uptime and load averages |
| `uname -a` | Display kernel and system information |
| `lsblk` | List block devices |

## 7. Logs

| Command | Purpose |
|---|---|
| `journalctl -b` | View logs from the current boot |
| `dmesg` | Inspect kernel messages |
| `tail -f /var/log/syslog` | Follow a log where that file exists |
| `grep -i "failed" file.log` | Search logs case-insensitively |

## Important Notes

- Command availability and log paths differ by distribution.
- Use `command --help` or `man command` to check syntax.
- Do not run destructive commands on a production system without a reviewed change plan.

