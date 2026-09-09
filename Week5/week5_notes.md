# IT 123 – System Administration
## Week 5 Laboratory Notes: System Initialization and Service Management

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course & Year:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## 1. Laboratory Overview & Objectives

In modern Linux operating systems, **systemd** serves as the init system (PID 1) responsible for bootstrapping user space, mounting filesystems, coordinating parallel service startup, and managing system state transitions through targets.

### Key Objectives:
1. Examine Linux initialization targets and runlevels via `systemctl`.
2. Manage service lifecycles (start, stop, restart, reload, enable, disable, inspect status).
3. Investigate service dependencies and diagnose service degradation using `journalctl`.
4. Configure production web (`apache2`) and FTP (`vsftpd`) service startup policies.
5. Contrast Linux service management with Microsoft Windows Server Service Control Manager (SCM).

---

## 2. Part 1: Exploring the Linux Boot Process (systemd Targets)

Traditional SysVinit runlevels (0–6) are replaced in systemd by **target units** (`.target`).

| Target Name | SysVinit Runlevel | Description |
| :--- | :--- | :--- |
| `poweroff.target` | Runlevel 0 | Shuts down and powers off the system. |
| `rescue.target` | Runlevel 1 | Single-user administrative maintenance mode; minimal services. |
| `multi-user.target` | Runlevel 3 | Multi-user text-mode environment with full networking (Default for Ubuntu Server). |
| `graphical.target` | Runlevel 5 | Multi-user environment with display manager and GUI desktop. |
| `reboot.target` | Runlevel 6 | Reboots the system. |

### Commands Executed:
```bash
# 1. Inspect Default Boot Target
systemctl get-default
# Expected output: multi-user.target

# 2. List All Active Target Units
systemctl list-units --type=target

# 3. Switching Targets (Isolation Mode)
# Switch to emergency/rescue mode:
sudo systemctl isolate rescue.target

# Return to standard multi-user mode:
sudo systemctl isolate multi-user.target
```

---

## 3. Part 2: Service Lifecycle Management

Services in systemd are defined by unit configuration files (`.service`) located in `/lib/systemd/system/` or `/etc/systemd/system/`.

### Commands Executed (SSH Service):
```bash
# 1. List currently running services
systemctl list-units --type=service --state=running

# 2. Inspect SSH daemon detailed status
systemctl status ssh --no-pager

# 3. Runtime Lifecycle Control
sudo systemctl stop ssh      # Halts the running daemon
sudo systemctl start ssh     # Starts the daemon
sudo systemctl restart ssh   # Stops and immediately restarts the daemon

# 4. Boot Startup Configuration (Symlink creation in /etc/systemd/system/multi-user.target.wants/)
sudo systemctl disable ssh   # Removes boot symlink (will NOT launch on reboot)
sudo systemctl enable ssh    # Creates boot symlink (will launch automatically on reboot)

# 5. Querying Boot Startup State
systemctl is-enabled ssh     # Returns: 'enabled' or 'disabled'
```

---

## 4. Part 3: Troubleshooting Services with Logs

When a service crashes or fails to start, `journalctl` queries the unified binary `systemd-journald` log buffer.

```bash
# 1. Stop service to simulate degradation
sudo systemctl stop ssh

# 2. Verify inactive state
systemctl status ssh --no-pager
# Output shows: Active: inactive (dead)

# 3. Query systemd journal filtered for ssh unit within recent timeframe
journalctl -u ssh --since "5 minutes ago" --no-pager

# 4. Restore service to healthy operation
sudo systemctl start ssh
systemctl is-active ssh
# Output shows: active
```

---

## 5. Part 4: Windows Server Comparison (PowerShell & services.msc)

In Windows Server, background daemons are managed by the **Service Control Manager (SCM)**.

```powershell
# List all running services
Get-Service | Where-Object {$_.Status -eq 'Running'}

# Inspect specific service (e.g., Spooler or W32Time)
Get-Service Spooler | Select-Object Name, DisplayName, Status, StartType

# Stop, Start, and Restart
Stop-Service -Name Spooler
Start-Service -Name Spooler
Restart-Service -Name Spooler

# Configure Startup Type (Automatic = Enabled, Disabled = Disabled, Manual = Manual)
Set-Service -Name Spooler -StartupType Automatic
Set-Service -Name Spooler -StartupType Disabled
```

---

## 6. Part 5: Student Exercise — Apache & FTP Production Scenario

### Scenario Requirements:
1. **Apache Web Server (`apache2`)**: Mission-critical service; must automatically start on every system boot.
2. **FTP Service (`vsftpd`)**: Infrequent maintenance service; must be **disabled** at boot to save resources and reduce attack surface, but available for manual invocation when needed.

### Implementation Script:
```bash
# 1. Update repository index and install both services
sudo apt update
sudo apt install apache2 vsftpd -y

# 2. Configure startup policies
sudo systemctl enable apache2    # Enable Apache at boot
sudo systemctl disable vsftpd   # Disable FTP at boot

# 3. Verify Boot Persistence
systemctl is-enabled apache2    # Expected: enabled
systemctl is-enabled vsftpd     # Expected: disabled

# 4. Verify Active Runtime States
systemctl status apache2 --no-pager
systemctl status vsftpd --no-pager
```

### Verification Results:
| Service Unit | Package Name | Startup Policy (`is-enabled`) | Runtime State | Operational Rationale |
| :--- | :--- | :--- | :--- | :--- |
| `apache2.service` | Apache 2.4 HTTP Server | **`enabled`** | `active (running)` | Enterprise web presence must sustain zero downtime upon host reboot. |
| `vsftpd.service` | Very Secure FTP Daemon | **`disabled`** | `inactive (dead)` / Manual | Unused listening ports present an unmonitored attack vector. Started on-demand. |

---

## 7. Technical Observations & Reflection

1. **Unit File Symlink Mechanics:** Running `systemctl enable <unit>` creates a symbolic link in `/etc/systemd/system/multi-user.target.wants/` pointing back to `/lib/systemd/system/<unit>.service`. Conversely, `systemctl disable` removes this link.
2. **Disabled vs. Masked:** A *disabled* service can still be started manually via `systemctl start`. If a service must be completely prevented from starting (even by dependencies or manual commands), it should be *masked* via `systemctl mask <unit>`, which symlinks the unit to `/dev/null`.
3. **Security Hardening:** Keeping auxiliary services like FTP disabled until required aligns with the Principle of Least Privilege and minimizes unnecessary network socket listeners.
