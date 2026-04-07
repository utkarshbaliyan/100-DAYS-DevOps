# 🐧 Linux — Complete DevOps Interview Notes

> **Covers:** Basics → Advanced | File System · Permissions · Networking · Process Management · Shell Scripting · System Admin · Security · Performance

---

## Table of Contents

1. [Linux Architecture & Kernel](#1-linux-architecture--kernel)
2. [File System Hierarchy](#2-file-system-hierarchy)
3. [Essential Commands](#3-essential-commands)
4. [File Permissions & Ownership](#4-file-permissions--ownership)
5. [Users & Groups](#5-users--groups)
6. [Process Management](#6-process-management)
7. [Package Management](#7-package-management)
8. [Networking](#8-networking)
9. [Storage & Disk Management](#9-storage--disk-management)
10. [Shell & Bash Scripting](#10-shell--bash-scripting)
11. [Systemd & Service Management](#11-systemd--service-management)
12. [Logging & Monitoring](#12-logging--monitoring)
13. [SSH & Remote Access](#13-ssh--remote-access)
14. [Security & Hardening](#14-security--hardening)
15. [Performance Tuning](#15-performance-tuning)
16. [Cron Jobs & Scheduling](#16-cron-jobs--scheduling)
17. [Environment Variables & Config](#17-environment-variables--config)
18. [Linux for DevOps — Advanced Topics](#18-linux-for-devops--advanced-topics)

---

## 1. Linux Architecture & Kernel

### Layers (Bottom to Top)
```
┌──────────────────────────────┐
│        User Applications     │  ← bash, vim, docker, etc.
├──────────────────────────────┤
│         Shell / CLI          │  ← bash, zsh, sh
├──────────────────────────────┤
│      System Libraries        │  ← glibc, libc
├──────────────────────────────┤
│         Linux Kernel         │  ← core of the OS
├──────────────────────────────┤
│           Hardware           │  ← CPU, RAM, Disk, NIC
└──────────────────────────────┘
```

### Kernel Responsibilities
- **Process scheduling** — decides which process gets CPU time
- **Memory management** — virtual memory, paging, swapping
- **Device drivers** — communicates with hardware
- **System calls** — interface between user space and kernel space
- **File system management** — ext4, xfs, btrfs, etc.

### Key Concepts
| Concept | Description |
|---|---|
| **Kernel Space** | Where the kernel runs with full hardware access |
| **User Space** | Where applications run with restricted access |
| **System Call** | API for user space to request kernel services |
| **Monolithic Kernel** | All core services run in kernel space (Linux) |
| **Microkernel** | Minimal kernel; services run in user space |

```bash
uname -r          # Current kernel version
uname -a          # All system info
cat /proc/version # Kernel details
lsmod             # List loaded kernel modules
modprobe <module> # Load a kernel module
```

---

## 2. File System Hierarchy

Linux follows **FHS (Filesystem Hierarchy Standard)**:

```
/
├── bin/      → Essential user binaries (ls, cp, mv)
├── sbin/     → System binaries (iptables, fdisk)
├── etc/      → Configuration files
├── home/     → User home directories
├── root/     → Root user's home
├── var/      → Variable data (logs, spool, cache)
├── tmp/      → Temporary files (cleared on reboot)
├── usr/      → User programs and data
│   ├── bin/  → Non-essential user binaries
│   ├── lib/  → Libraries
│   └── local/→ Locally compiled software
├── lib/      → Essential shared libraries
├── dev/      → Device files (sda, tty, null)
├── proc/     → Virtual FS: process/kernel info
├── sys/      → Virtual FS: hardware/driver info
├── mnt/      → Temporary mount points
├── media/    → Removable media (USB, CD)
├── opt/      → Optional/third-party software
├── boot/     → Bootloader, kernel images
└── srv/      → Data for services (web, ftp)
```

### Important Files
| File | Purpose |
|---|---|
| `/etc/passwd` | User account info |
| `/etc/shadow` | Encrypted passwords |
| `/etc/group` | Group definitions |
| `/etc/hosts` | Static hostname resolution |
| `/etc/resolv.conf` | DNS server configuration |
| `/etc/fstab` | File system mount table |
| `/etc/crontab` | System-wide cron jobs |
| `/proc/meminfo` | Memory usage info |
| `/proc/cpuinfo` | CPU info |
| `/proc/net/dev` | Network interface stats |

---

## 3. Essential Commands

### Navigation
```bash
pwd               # Print current directory
ls -la            # List all files with details
ls -lh            # Human-readable sizes
cd /path/to/dir   # Change directory
cd ~              # Go to home directory
cd -              # Go to previous directory
```

### File Operations
```bash
touch file.txt           # Create empty file / update timestamp
cp src.txt dst.txt       # Copy file
cp -r src/ dst/          # Copy directory recursively
mv old.txt new.txt       # Move/rename file
rm file.txt              # Delete file
rm -rf dir/              # Force delete directory (use carefully!)
mkdir -p a/b/c           # Create nested directories
ln -s /real/path link    # Create symbolic (soft) link
ln /real/path hardlink   # Create hard link
```

### Viewing Files
```bash
cat file.txt             # Print file content
less file.txt            # Paginated view (q to quit)
head -n 20 file.txt      # First 20 lines
tail -n 20 file.txt      # Last 20 lines
tail -f /var/log/syslog  # Follow file in real time
wc -l file.txt           # Count lines
```

### Searching
```bash
find / -name "*.conf"            # Find files by name
find /var -mtime -7              # Files modified last 7 days
find . -type f -size +100M       # Files larger than 100MB
grep "error" /var/log/syslog     # Search for pattern
grep -r "TODO" /project/         # Recursive search
grep -i "error" file.txt         # Case-insensitive search
grep -n "pattern" file.txt       # Show line numbers
grep -v "pattern" file.txt       # Invert match (exclude)
```

### Text Processing
```bash
awk '{print $1}' file.txt           # Print first column
awk -F: '{print $1}' /etc/passwd    # Split by colon, print field 1
sed 's/old/new/g' file.txt          # Replace text (in output)
sed -i 's/old/new/g' file.txt       # Replace text (in-place)
cut -d: -f1 /etc/passwd             # Cut field by delimiter
sort file.txt                        # Sort lines
sort -k2 -n file.txt                 # Sort by column 2 numerically
uniq file.txt                        # Remove consecutive duplicates
uniq -c file.txt                     # Count occurrences
tr 'a-z' 'A-Z' < file.txt           # Translate characters
```

### Archiving & Compression
```bash
tar -czf archive.tar.gz dir/         # Create gzipped tarball
tar -xzf archive.tar.gz              # Extract gzipped tarball
tar -cjf archive.tar.bz2 dir/        # Create bzip2 archive
tar -tzf archive.tar.gz              # List contents without extracting
gzip file.txt                         # Compress file (replaces original)
gunzip file.txt.gz                    # Decompress
zip -r archive.zip dir/              # Zip a directory
unzip archive.zip                     # Unzip
```

### Disk Usage
```bash
df -h                  # Disk space usage (human-readable)
du -sh /var/log/       # Size of a directory
du -sh * | sort -h     # Sort directories by size
```

### Pipes & Redirection
```bash
command > file.txt      # Redirect stdout to file (overwrite)
command >> file.txt     # Redirect stdout to file (append)
command 2> error.txt    # Redirect stderr
command &> all.txt      # Redirect both stdout and stderr
command < input.txt     # Redirect stdin
cmd1 | cmd2             # Pipe stdout of cmd1 to cmd2
cmd1 | tee file.txt | cmd2   # Pipe and also save to file
```

---

## 4. File Permissions & Ownership

### Permission Structure
```
-rwxr-xr--  1  alice  devs  4096  Jan 1  file.sh
│└┬┘└┬┘└┬┘      │      │
│ │   │   │      │      └─ Group
│ │   │   │      └─ Owner
│ │   │   └─ Others permissions (r--)
│ │   └─ Group permissions (r-x)
│ └─ Owner permissions (rwx)
└─ File type (- = file, d = dir, l = symlink)
```

### Permission Values
| Symbol | Octal | Meaning |
|---|---|---|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `-` | 0 | No permission |

**Examples:**
```
rwx = 7  (4+2+1)
rw- = 6  (4+2)
r-x = 5  (4+1)
r-- = 4  (4)
--- = 0
```

### chmod
```bash
chmod 755 file.sh        # rwxr-xr-x (common for executables)
chmod 644 file.txt       # rw-r--r-- (common for files)
chmod 600 secret.key     # rw------- (private key)
chmod 777 file           # rwxrwxrwx (avoid in production!)
chmod +x script.sh       # Add execute for all
chmod u+x,g-w file       # Add execute for user, remove write from group
chmod -R 755 /var/www    # Recursive
```

### chown & chgrp
```bash
chown alice file.txt          # Change owner
chown alice:devs file.txt     # Change owner and group
chown -R alice:devs /project  # Recursive
chgrp devs file.txt           # Change group only
```

### Special Permissions
| Permission | Octal | Effect |
|---|---|---|
| **SUID** (Set User ID) | 4000 | Executable runs as the file's owner |
| **SGID** (Set Group ID) | 2000 | Files created inherit the directory's group |
| **Sticky Bit** | 1000 | Only file owner can delete in a shared directory |

```bash
chmod u+s /usr/bin/passwd    # Set SUID
chmod g+s /shared/dir        # Set SGID
chmod +t /tmp                # Set sticky bit
chmod 4755 file              # SUID + 755
ls -l /tmp                   # drwxrwxrwt — sticky bit shown as 't'
```

### umask
```bash
umask          # Show current umask (e.g., 022)
umask 027      # Set umask: new files get 640, dirs get 750
# umask is subtracted from 666 (files) or 777 (dirs)
# 666 - 022 = 644 (default file permissions)
```

---

## 5. Users & Groups

### User Management
```bash
useradd alice                  # Create user (no home dir on some distros)
useradd -m -s /bin/bash alice  # Create with home dir and bash shell
usermod -aG sudo alice         # Add user to sudo group
usermod -s /bin/zsh alice      # Change shell
userdel alice                  # Delete user
userdel -r alice               # Delete user + home directory
passwd alice                   # Set/change password
passwd -l alice                # Lock account
passwd -u alice                # Unlock account
```

### Group Management
```bash
groupadd devs           # Create group
groupdel devs           # Delete group
gpasswd -a alice devs   # Add user to group
gpasswd -d alice devs   # Remove user from group
groups alice            # Show groups for user
id alice                # UID, GID, groups
```

### Switching Users
```bash
su alice                # Switch to user (requires password)
su - alice              # Switch with login environment
sudo command            # Run command as root
sudo -u alice command   # Run as specific user
sudo -i                 # Start root shell
whoami                  # Current user
```

### /etc/passwd format
```
username:x:UID:GID:GECOS:home_dir:shell
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
```

### /etc/sudoers (via visudo)
```bash
# Allow alice to run all commands as root
alice ALL=(ALL:ALL) ALL

# Allow passwordless sudo
alice ALL=(ALL) NOPASSWD: ALL

# Allow specific command
alice ALL=(ALL) /usr/bin/systemctl restart nginx
```

---

## 6. Process Management

### Viewing Processes
```bash
ps aux                    # All processes (BSD style)
ps -ef                    # All processes (UNIX style)
ps aux | grep nginx       # Find specific process
top                       # Real-time process viewer
htop                      # Enhanced top (install separately)
pgrep nginx               # Get PID by name
pidof nginx               # Get PID(s) of a program
```

### Process Control
```bash
kill <PID>                # Send SIGTERM (graceful)
kill -9 <PID>             # Send SIGKILL (force kill)
kill -HUP <PID>           # Hangup (reload config)
killall nginx             # Kill all processes named nginx
pkill -f "python script"  # Kill by pattern
```

### Signals Reference
| Signal | Number | Use |
|---|---|---|
| SIGTERM | 15 | Graceful termination (default `kill`) |
| SIGKILL | 9 | Force kill (cannot be caught) |
| SIGHUP | 1 | Hangup / reload config |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGSTOP | 19 | Pause process |
| SIGCONT | 18 | Resume paused process |

### Background & Foreground Jobs
```bash
command &           # Run in background
jobs                # List background jobs
fg %1               # Bring job 1 to foreground
bg %1               # Send job 1 to background
Ctrl+Z              # Pause current process
Ctrl+C              # Interrupt current process
nohup command &     # Run immune to hangup (survives logout)
disown %1           # Remove job from shell's job list
```

### Process Priority (Nice)
```bash
nice -n 10 command          # Start with low priority (nice=10)
nice -n -10 command         # Start with high priority (needs root)
renice -n 5 -p <PID>        # Change priority of running process
# Nice values: -20 (highest) to 19 (lowest), default 0
```

### /proc filesystem
```bash
ls /proc/<PID>/            # Process directory
cat /proc/<PID>/status     # Process status
cat /proc/<PID>/cmdline    # Command that started the process
cat /proc/<PID>/environ    # Environment variables
cat /proc/<PID>/fd/        # Open file descriptors
```

---

## 7. Package Management

### Debian/Ubuntu (APT)
```bash
apt update                        # Update package index
apt upgrade                       # Upgrade installed packages
apt install nginx                 # Install package
apt remove nginx                  # Remove (keep config)
apt purge nginx                   # Remove + config files
apt autoremove                    # Remove unused dependencies
apt search nginx                  # Search packages
apt show nginx                    # Package details
dpkg -l                           # List installed packages
dpkg -i package.deb               # Install .deb file
dpkg -r package                   # Remove package
dpkg -L nginx                     # List files from package
```

### RHEL/CentOS/Fedora (YUM/DNF)
```bash
yum update                        # Update all packages
yum install nginx                 # Install package
yum remove nginx                  # Remove package
yum search nginx                  # Search
yum info nginx                    # Package info
dnf install nginx                 # DNF (newer, Fedora/RHEL 8+)
rpm -ivh package.rpm              # Install .rpm file
rpm -qa                           # List all installed packages
rpm -ql nginx                     # List files from package
```

---

## 8. Networking

### IP & Interface Commands
```bash
ip addr show             # Show IP addresses (modern)
ip addr show eth0        # Show specific interface
ip link show             # Show network interfaces
ip route show            # Show routing table
ip route add default via 192.168.1.1   # Add default route
ifconfig                 # Legacy (net-tools)
hostname                 # Show hostname
hostname -I              # Show all IPs
```

### Connectivity Testing
```bash
ping google.com                   # ICMP ping
ping -c 4 google.com              # Limit to 4 pings
traceroute google.com             # Trace network path
tracepath google.com              # Alternative trace
mtr google.com                    # Real-time traceroute
curl -I https://example.com       # HTTP headers only
curl -o file.txt https://url      # Download file
wget https://url/file.zip         # Download file
```

### DNS
```bash
nslookup google.com               # DNS lookup
dig google.com                    # Detailed DNS lookup
dig google.com MX                 # MX records
dig @8.8.8.8 google.com          # Query specific DNS server
host google.com                   # Simple DNS lookup
cat /etc/resolv.conf              # DNS server config
cat /etc/hosts                    # Static host entries
```

### Ports & Connections
```bash
netstat -tulpn                    # All listening ports + PID
ss -tulpn                         # Modern netstat replacement
ss -s                             # Socket summary
lsof -i :80                       # What's using port 80
lsof -i tcp                       # All TCP connections
nc -zv host 80                    # Test if port is open
telnet host 80                    # Test connectivity
nmap -sS 192.168.1.0/24          # Network scan
nmap -p 22,80,443 host           # Scan specific ports
```

### Firewall (iptables)
```bash
iptables -L -n -v                 # List all rules
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # Allow port 80
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # Allow SSH
iptables -A INPUT -j DROP                         # Drop all other input
iptables -D INPUT 1               # Delete rule 1 from INPUT chain
iptables-save > /etc/iptables/rules.v4  # Save rules
iptables-restore < /etc/iptables/rules.v4  # Restore rules
```

### Firewall (firewalld — RHEL/CentOS)
```bash
firewall-cmd --state                              # Check status
firewall-cmd --list-all                           # List all rules
firewall-cmd --add-service=http --permanent      # Allow HTTP
firewall-cmd --add-port=8080/tcp --permanent     # Allow port 8080
firewall-cmd --remove-service=http --permanent   # Remove HTTP
firewall-cmd --reload                             # Apply changes
```

### UFW (Ubuntu Firewall)
```bash
ufw enable / disable              # Enable or disable UFW
ufw status verbose                # Show status
ufw allow 22/tcp                  # Allow SSH
ufw allow http                    # Allow HTTP
ufw deny 23                       # Deny telnet
ufw delete allow 22/tcp           # Remove rule
```

### Network Configuration Files
```bash
/etc/network/interfaces     # Debian — static IP config
/etc/netplan/*.yaml         # Ubuntu 18+ — Netplan config
/etc/sysconfig/network-scripts/ifcfg-eth0  # CentOS/RHEL
```

---

## 9. Storage & Disk Management

### Viewing Disks & Partitions
```bash
lsblk                           # Block devices tree
fdisk -l                        # Disk partition info
parted -l                       # Partition info (GPT-aware)
blkid                           # UUID and file system type of devices
df -hT                          # Disk usage with filesystem type
mount                           # Show mounted filesystems
```

### Partitioning
```bash
fdisk /dev/sdb                  # Interactive partition editor (MBR)
parted /dev/sdb                 # Parted (supports GPT + MBR)
  mklabel gpt                   # Create GPT partition table
  mkpart primary ext4 0% 100%   # Create partition
```

### Filesystems
```bash
mkfs.ext4 /dev/sdb1             # Format as ext4
mkfs.xfs /dev/sdb1              # Format as XFS
mkfs.btrfs /dev/sdb1            # Format as Btrfs
tune2fs -l /dev/sda1            # Show ext4 filesystem info
xfs_info /dev/sda1              # XFS info
fsck /dev/sdb1                  # Check & repair filesystem (unmounted!)
e2fsck -f /dev/sdb1             # Force check ext4
```

### Mounting
```bash
mount /dev/sdb1 /mnt/data       # Mount partition
mount -t ext4 /dev/sdb1 /data   # Mount with filesystem type
umount /mnt/data                # Unmount
umount -l /mnt/data             # Lazy unmount (busy filesystem)
```

### /etc/fstab (Persistent Mounts)
```
# Device         MountPoint   FSType  Options       dump  pass
UUID=xxxx-xxxx   /mnt/data    ext4    defaults      0     2
/dev/sdb1        /mnt/backup  xfs     defaults,nofail 0   2
tmpfs            /tmp         tmpfs   defaults,size=1G 0  0
```

### LVM (Logical Volume Manager)
```bash
# Physical Volumes
pvcreate /dev/sdb1 /dev/sdc1    # Initialize PVs
pvs / pvdisplay                  # Show PVs

# Volume Groups
vgcreate vg_data /dev/sdb1      # Create VG
vgextend vg_data /dev/sdc1      # Extend VG
vgs / vgdisplay                  # Show VGs

# Logical Volumes
lvcreate -L 50G -n lv_data vg_data   # Create 50GB LV
lvcreate -l 100%FREE -n lv_data vg_data  # Use all free space
lvextend -L +20G /dev/vg_data/lv_data    # Extend LV
resize2fs /dev/vg_data/lv_data           # Resize ext4 filesystem
xfs_growfs /mnt/data                     # Resize XFS filesystem
lvs / lvdisplay                           # Show LVs
```

### Swap
```bash
swapon --show                   # Show active swap
mkswap /dev/sdb2                # Format swap partition
swapon /dev/sdb2                # Enable swap
swapoff /dev/sdb2               # Disable swap
# Create swap file:
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

---

## 10. Shell & Bash Scripting

### Shebang & Basics
```bash
#!/bin/bash          # Shebang — tells OS which interpreter to use
#!/usr/bin/env bash  # Portable shebang
chmod +x script.sh   # Make script executable
./script.sh          # Run script
bash script.sh       # Run with bash explicitly
```

### Variables
```bash
name="Alice"          # Assign (no spaces around =)
echo $name            # Use variable
echo "${name}_backup" # Curly braces for clarity
readonly PI=3.14      # Constant variable
unset name            # Delete variable

# Special variables
$0    # Script name
$1-$9 # Positional arguments
$#    # Number of arguments
$@    # All arguments (separate words)
$*    # All arguments (single string)
$?    # Exit code of last command
$$    # Current script's PID
$!    # PID of last background process
```

### Strings
```bash
str="Hello World"
echo ${#str}            # Length: 11
echo ${str:6}           # Substring: "World"
echo ${str:0:5}         # Substring: "Hello"
echo ${str/World/Linux} # Replace: "Hello Linux"
echo ${str,,}           # Lowercase
echo ${str^^}           # Uppercase
echo ${str#Hello }      # Remove prefix
echo ${str%World}       # Remove suffix
```

### Arrays
```bash
fruits=("apple" "banana" "cherry")
echo ${fruits[0]}       # apple
echo ${fruits[@]}       # All elements
echo ${#fruits[@]}      # Length: 3
fruits+=("mango")       # Append
unset fruits[1]         # Delete element
for f in "${fruits[@]}"; do echo $f; done
```

### Conditionals
```bash
# if/elif/else
if [ "$name" == "Alice" ]; then
  echo "Hello Alice"
elif [ "$name" == "Bob" ]; then
  echo "Hello Bob"
else
  echo "Unknown"
fi

# File tests
[ -f file.txt ]   # File exists and is regular file
[ -d /tmp ]       # Directory exists
[ -e path ]       # Path exists (any type)
[ -r file ]       # Readable
[ -w file ]       # Writable
[ -x file ]       # Executable
[ -s file ]       # File is non-empty
[ -z "$var" ]     # String is empty
[ -n "$var" ]     # String is non-empty

# Numeric comparisons
[ $a -eq $b ]     # Equal
[ $a -ne $b ]     # Not equal
[ $a -lt $b ]     # Less than
[ $a -gt $b ]     # Greater than
[ $a -le $b ]     # Less or equal
[ $a -ge $b ]     # Greater or equal

# Use [[ ]] for extended tests (pattern matching, &&, ||)
[[ $name == A* ]]     # Pattern match
[[ -f file && -r file ]]  # AND
[[ -f f1 || -f f2 ]]      # OR
```

### Loops
```bash
# for loop
for i in 1 2 3 4 5; do echo $i; done
for i in {1..10}; do echo $i; done
for ((i=0; i<10; i++)); do echo $i; done
for file in /var/log/*.log; do echo $file; done

# while loop
count=0
while [ $count -lt 5 ]; do
  echo "Count: $count"
  ((count++))
done

# until loop (runs until condition is true)
until [ $count -eq 10 ]; do
  ((count++))
done

# Loop control
break       # Exit loop
continue    # Skip to next iteration
```

### Functions
```bash
greet() {
  local name="$1"    # local scope
  echo "Hello, $name!"
  return 0           # exit code
}

greet "Alice"
result=$(greet "Bob")   # Capture output
```

### Error Handling
```bash
set -e          # Exit on any error
set -u          # Exit on undefined variable
set -o pipefail # Exit if any pipe command fails
set -x          # Debug mode (print each command)

# Trap errors
trap 'echo "Error on line $LINENO"; exit 1' ERR
trap 'cleanup; exit' EXIT   # Run cleanup on exit

# Check command success
if ! command -v docker &>/dev/null; then
  echo "docker not found"
  exit 1
fi
```

### Practical Script Template
```bash
#!/bin/bash
set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
readonly LOG_FILE="/var/log/myscript.log"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"; }
error() { log "ERROR: $*" >&2; exit 1; }

usage() {
  echo "Usage: $0 [options] <argument>"
  echo "  -h  Show this help"
  exit 0
}

# Parse flags
while getopts "h" opt; do
  case $opt in
    h) usage ;;
    *) error "Unknown option: $opt" ;;
  esac
done

main() {
  log "Script started"
  # Your logic here
  log "Script finished"
}

main "$@"
```

### Here Documents (heredoc)
```bash
cat <<EOF > config.txt
server {
  listen 80;
  server_name example.com;
}
EOF

# Suppress leading tabs:
cat <<-EOF
	Indented content
EOF
```

---

## 11. Systemd & Service Management

### Service Management
```bash
systemctl start nginx           # Start service
systemctl stop nginx            # Stop service
systemctl restart nginx         # Restart service
systemctl reload nginx          # Reload config (no downtime)
systemctl status nginx          # Status + recent logs
systemctl enable nginx          # Start on boot
systemctl disable nginx         # Don't start on boot
systemctl is-active nginx       # active / inactive
systemctl is-enabled nginx      # enabled / disabled
systemctl list-units --type=service  # List all services
systemctl list-units --failed        # Show failed services
```

### Systemd Unit File
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target
Requires=postgresql.service

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp --config /etc/myapp/config.yml
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload          # Reload unit files after changes
journalctl -u nginx              # Logs for nginx service
journalctl -u nginx -f           # Follow logs
journalctl -u nginx --since "1 hour ago"
journalctl -p err                # Show only errors
```

### Service Types
| Type | Description |
|---|---|
| `simple` | ExecStart is the main process |
| `forking` | Process forks; parent exits |
| `oneshot` | Runs once then exits |
| `notify` | Process sends ready notification |
| `idle` | Starts after other jobs finish |

### Targets (Runlevels)
| Target | Equivalent | Description |
|---|---|---|
| `poweroff.target` | 0 | Shutdown |
| `rescue.target` | 1 | Single-user mode |
| `multi-user.target` | 3 | Multi-user, no GUI |
| `graphical.target` | 5 | Multi-user + GUI |
| `reboot.target` | 6 | Reboot |

```bash
systemctl get-default                          # Show current target
systemctl set-default multi-user.target        # Set default target
systemctl isolate rescue.target                # Switch immediately
```

---

## 12. Logging & Monitoring

### System Logs
```bash
journalctl                        # All systemd logs
journalctl -f                     # Follow live
journalctl -n 100                 # Last 100 lines
journalctl --since "2024-01-01"
journalctl --since "1 hour ago"
journalctl -p err                 # Filter by priority (emerg,alert,crit,err,warning,notice,info,debug)
journalctl -u sshd -f             # Service-specific follow

# Traditional log files
tail -f /var/log/syslog           # General system log (Debian)
tail -f /var/log/messages         # General system log (RHEL)
tail -f /var/log/auth.log         # Authentication log
tail -f /var/log/nginx/access.log # Nginx access log
/var/log/kern.log                 # Kernel messages
dmesg                             # Kernel ring buffer
dmesg | grep error                # Filter errors
```

### System Monitoring
```bash
top               # Real-time processes (q to quit)
htop              # Enhanced top
vmstat 1 5        # VM/CPU/IO stats every 1s, 5 times
iostat -x 1       # Disk I/O stats
sar -u 1 5        # CPU usage history
free -h           # Memory usage
uptime            # Load averages
w                 # Who is logged in + load
```

### Load Average
```bash
uptime
# output: 14:30:12 up 5 days, 3 load averages: 0.52, 0.45, 0.40
#                                               1min  5min  15min
# Rule: load > number_of_CPUs means system is overloaded
nproc             # Number of CPUs
cat /proc/cpuinfo | grep processor | wc -l
```

### Memory
```bash
free -h
# Output:
#              total   used   free   shared   buff/cache   available
# Mem:          15G    3.2G   8.1G    512M       3.7G        11G

# available = free + reclaimable buff/cache
# buff/cache = can be freed when needed
cat /proc/meminfo
```

---

## 13. SSH & Remote Access

### SSH Basics
```bash
ssh user@host                           # Connect
ssh -p 2222 user@host                   # Custom port
ssh -i ~/.ssh/id_rsa user@host          # Use specific key
ssh -L 8080:localhost:80 user@host      # Local port forward
ssh -R 9090:localhost:3000 user@host    # Remote port forward
ssh -D 1080 user@host                   # SOCKS5 proxy
ssh -N -f -L 8080:localhost:80 user@host  # Background tunnel
```

### Key Management
```bash
ssh-keygen -t ed25519 -C "email@example.com"  # Generate Ed25519 key (recommended)
ssh-keygen -t rsa -b 4096                      # Generate RSA key
ssh-copy-id user@host                          # Copy public key to server
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys  # Manual way

# Key files
~/.ssh/id_ed25519         # Private key (NEVER share)
~/.ssh/id_ed25519.pub     # Public key (safe to share)
~/.ssh/authorized_keys    # Authorized keys on server
~/.ssh/known_hosts        # Known host fingerprints
~/.ssh/config             # SSH client config
```

### SSH Config (~/.ssh/config)
```
Host myserver
  HostName 192.168.1.100
  User alice
  Port 2222
  IdentityFile ~/.ssh/id_ed25519
  ServerAliveInterval 60

Host bastion
  HostName bastion.example.com
  User ec2-user

Host internal-server
  HostName 10.0.0.5
  ProxyJump bastion
```

### SSH Server Config (/etc/ssh/sshd_config)
```
Port 22
PermitRootLogin no            # Disable root login
PasswordAuthentication no     # Disable password auth (key-only)
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
ClientAliveInterval 300
AllowUsers alice bob
```

```bash
systemctl restart sshd        # Apply sshd changes
sshd -t                       # Test config syntax
```

### SCP & SFTP
```bash
scp file.txt user@host:/remote/path/    # Upload file
scp user@host:/remote/file.txt ./       # Download file
scp -r dir/ user@host:/remote/          # Upload directory
sftp user@host                          # Interactive SFTP
rsync -avz local/ user@host:/remote/    # Sync with rsync
rsync -avz --delete local/ user@host:/remote/  # Mirror (delete extra)
rsync -avz -e "ssh -p 2222" local/ user@host:/remote/  # Custom port
```

---

## 14. Security & Hardening

### File Integrity
```bash
sha256sum file.txt              # Compute SHA256 hash
sha256sum -c checksums.txt      # Verify checksums
md5sum file.txt                 # MD5 (weak, avoid for security)
```

### SELinux (Security-Enhanced Linux)
```bash
getenforce                      # Enforcing / Permissive / Disabled
setenforce 0                    # Set Permissive (temporary)
setenforce 1                    # Set Enforcing (temporary)
# Permanent: edit /etc/selinux/config → SELINUX=enforcing
sestatus                        # Detailed status
ls -Z /var/www/html             # Show SELinux context
chcon -t httpd_sys_content_t /var/www/html/index.html  # Change context
restorecon -Rv /var/www/html    # Restore default context
audit2allow -a                  # Generate policy from AVC denials
```

### AppArmor (Ubuntu/Debian)
```bash
aa-status                       # Show AppArmor status
aa-enforce /etc/apparmor.d/usr.sbin.nginx  # Enforce profile
aa-complain profile             # Set to complain mode
```

### fail2ban
```bash
systemctl status fail2ban
fail2ban-client status          # Show jails
fail2ban-client status sshd     # SSH jail status
fail2ban-client set sshd unbanip 1.2.3.4  # Unban IP
# Config: /etc/fail2ban/jail.local
```

### Auditing
```bash
last                            # Login history
lastb                           # Failed login attempts
who                             # Currently logged-in users
w                               # Who + what they're doing
auditctl -l                     # List audit rules
aureport --failed               # Failed audit events
ausearch -m USER_AUTH           # Authentication events
```

### File Attributes
```bash
lsattr file.txt                 # List attributes
chattr +i file.txt              # Immutable (can't be deleted/modified)
chattr -i file.txt              # Remove immutable
chattr +a file.txt              # Append-only
```

---

## 15. Performance Tuning

### CPU
```bash
lscpu                           # CPU architecture info
cat /proc/cpuinfo               # Detailed CPU info
mpstat -P ALL 1                 # Per-CPU stats
taskset -c 0,1 command          # Pin process to CPU 0 and 1
```

### Memory
```bash
cat /proc/meminfo               # Memory info
sysctl vm.swappiness            # Default 60 (0=prefer RAM, 100=prefer swap)
sysctl -w vm.swappiness=10      # Reduce swappiness (better for servers)
echo 3 > /proc/sys/vm/drop_caches  # Clear page cache (use with caution)
```

### Disk I/O
```bash
iostat -xm 2                    # Disk I/O stats (megabytes)
iotop                           # Top for disk I/O
hdparm -t /dev/sda              # Disk read speed test
dd if=/dev/zero of=test bs=1M count=1000  # Write speed test
```

### Network
```bash
iperf3 -s                       # Start iperf server
iperf3 -c server_ip             # Run bandwidth test
ethtool eth0                    # NIC info and speed
```

### sysctl — Kernel Parameters
```bash
sysctl -a                       # Show all parameters
sysctl net.core.somaxconn       # Max socket backlog
sysctl -w net.core.somaxconn=65535  # Set temporarily
# Permanent: echo "net.core.somaxconn = 65535" >> /etc/sysctl.conf
sysctl -p                       # Apply sysctl.conf

# Common tuning parameters
net.ipv4.tcp_syncookies = 1       # SYN flood protection
net.core.rmem_max = 16777216      # Max receive buffer
net.core.wmem_max = 16777216      # Max send buffer
vm.swappiness = 10                # Reduce swap usage
fs.file-max = 2097152             # Max open files
```

### Open File Limits
```bash
ulimit -n                       # Current open file limit
ulimit -n 65535                 # Set limit (temporary)
# Permanent (in /etc/security/limits.conf):
# * soft nofile 65535
# * hard nofile 65535
```

---

## 16. Cron Jobs & Scheduling

### Crontab Syntax
```
# ┌─── minute (0-59)
# │ ┌─── hour (0-23)
# │ │ ┌─── day of month (1-31)
# │ │ │ ┌─── month (1-12)
# │ │ │ │ ┌─── day of week (0-7, 0=Sun, 7=Sun)
# │ │ │ │ │
# * * * * *  command
```

### Examples
```bash
# Every minute
* * * * * /path/to/script.sh

# Every 5 minutes
*/5 * * * * /path/to/script.sh

# Daily at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# First day of each month at midnight
0 0 1 * * /path/to/script.sh

# Every 6 hours
0 */6 * * * /path/to/script.sh

# Redirect output to avoid email
* * * * * /script.sh >> /var/log/script.log 2>&1
```

### Crontab Commands
```bash
crontab -e              # Edit current user's crontab
crontab -l              # List current user's crontab
crontab -r              # Remove current user's crontab
crontab -u alice -l     # List alice's crontab (root only)
cat /etc/crontab        # System-wide crontab
ls /etc/cron.d/         # Drop-in cron files
ls /etc/cron.daily/     # Daily scripts
ls /etc/cron.hourly/    # Hourly scripts
```

### at — One-time Scheduling
```bash
at 2:30 PM                   # Schedule interactive command
at 2:30 PM tomorrow
at now + 2 hours
atq                          # List pending jobs
atrm 1                       # Remove job 1
```

---

## 17. Environment Variables & Config

### Variables
```bash
env                         # List all environment variables
printenv PATH               # Print specific variable
export VAR=value            # Set and export to child processes
export PATH=$PATH:/new/path # Append to PATH
echo $HOME $USER $SHELL     # Common vars

# Common environment variables
HOME     → /home/alice
USER     → alice
SHELL    → /bin/bash
PATH     → directories to search for executables
PWD      → current directory
LANG     → language/locale
EDITOR   → default text editor
```

### Persistence
```bash
~/.bashrc          # User-specific, non-login (interactive shells)
~/.bash_profile    # User-specific, login shells
~/.profile         # Fallback for login shells (POSIX)
/etc/environment   # System-wide variables (simple key=value)
/etc/profile       # System-wide for login shells
/etc/profile.d/    # Drop-in scripts for login shells

# Apply changes without relogin:
source ~/.bashrc
. ~/.bashrc          # Same thing
```

### Aliases
```bash
alias ll='ls -la'
alias grep='grep --color=auto'
alias k='kubectl'
# Put in ~/.bashrc for persistence
unalias ll          # Remove alias
```

---

## 18. Linux for DevOps — Advanced Topics

### Namespaces & Cgroups (Container Foundation)
```bash
# Linux namespaces isolate resources:
# PID, Network, Mount, UTS, IPC, User, Cgroup

# View process namespaces
ls -la /proc/<PID>/ns/

# cgroups — limit/track resource usage
cat /sys/fs/cgroup/memory/memory.limit_in_bytes
systemd-cgls               # Tree of cgroups
systemctl set-property nginx.service MemoryLimit=512M
```

### Strace & Debugging
```bash
strace ls                           # Trace system calls
strace -p <PID>                     # Attach to running process
strace -e open,read ls              # Filter specific calls
lsof -p <PID>                       # Open files by PID
lsof /var/log/syslog                # Who has this file open
```

### Kernel Tuning for Production
```bash
# /etc/sysctl.conf production values
net.ipv4.ip_forward = 1              # Required for Docker/K8s routing
net.bridge.bridge-nf-call-iptables = 1  # Required for Kubernetes
vm.overcommit_memory = 1             # Required for Redis
kernel.panic = 10                    # Reboot 10s after kernel panic
net.ipv4.tcp_tw_reuse = 1            # Reuse TIME_WAIT sockets
```

### Signals & Graceful Deployments
```bash
# Zero-downtime restart pattern
kill -HUP $(pidof nginx)        # Reload config without dropping connections
kill -USR2 $(pidof nginx)       # Upgrade binary gracefully
```

### inotify — File System Events
```bash
inotifywait -m /etc/nginx/      # Watch for file changes
inotifywait -r -e modify,create,delete /var/www/  # Recursive watch
```

### /proc & /sys Deep Dive
```bash
cat /proc/sys/fs/file-nr        # Open file handles / max
cat /proc/net/sockstat          # Socket statistics
cat /proc/loadavg               # Load average raw
echo 1 > /proc/sys/net/ipv4/ip_forward  # Enable IP forwarding
```

### Bash Performance Tips
```bash
# Use [[ ]] over [ ] for faster conditionals
# Use $(command) over `command` for readability
# Use printf over echo for portability
# Avoid forks: use string ops instead of external commands
# Use arrays instead of repeated string splits
```

### Useful One-Liners for DevOps
```bash
# Find and kill zombie processes
ps aux | awk '$8=="Z" {print $2}' | xargs kill -9

# Watch a command every 2 seconds
watch -n 2 'df -h'

# Check open file descriptors for a process
ls /proc/<PID>/fd | wc -l

# List top 10 largest files
find / -type f -printf '%s %p\n' | sort -rn | head -10

# Check which process is using a port
ss -tulpn | grep ':80'

# Monitor network connections count
watch -n 1 'ss -s'

# Test HTTP response time
curl -o /dev/null -s -w "Time: %{time_total}s\n" https://example.com

# Count unique IPs in nginx access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Find files changed in last 24 hours
find /etc -newer /etc/hostname -type f

# Base64 encode/decode
echo "secret" | base64
echo "c2VjcmV0Cg==" | base64 -d
```

---

## Quick Reference Cheat Sheet

### Most Used Commands
| Task | Command |
|---|---|
| Disk usage | `df -h` / `du -sh *` |
| Memory | `free -h` |
| CPU / Processes | `top` / `htop` |
| Network ports | `ss -tulpn` |
| Find file | `find / -name filename` |
| Search content | `grep -r "pattern" /path` |
| Service logs | `journalctl -u service -f` |
| Who's logged in | `w` / `who` |
| Load average | `uptime` |
| File permissions | `ls -la` |
| Running processes | `ps aux` |
| Kill process | `kill -9 <PID>` |
| Mount info | `lsblk` / `df -hT` |

### Exit Codes
| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of command |
| 126 | Command not executable |
| 127 | Command not found |
| 128+n | Killed by signal n |
| 130 | Killed by Ctrl+C (SIGINT) |

### File Permission Quick Reference
| chmod | Meaning | Use Case |
|---|---|---|
| 777 | rwxrwxrwx | Avoid — world writable |
| 755 | rwxr-xr-x | Executables, dirs |
| 644 | rw-r--r-- | Regular files |
| 600 | rw------- | Private keys, secrets |
| 640 | rw-r----- | Group readable configs |
| 400 | r-------- | Read-only private files |

---

*Notes compiled for DevOps interview preparation. Review `man <command>` for full documentation on any command.*
