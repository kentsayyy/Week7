# IT 123 – System Administration
## Week 2 Laboratory Notes: Installing and Configuring Windows & Linux Virtual Machines in VirtualBox

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course & Year:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## 1. Laboratory Overview & Objectives

The purpose of this laboratory is to establish a standardized, virtualized server laboratory environment using Oracle VM VirtualBox. This dual-server testbed (Windows Server and Ubuntu Server) forms the foundation for upcoming enterprise system administration activities, including directory services, network infrastructure (DNS/DHCP), security hardening, and service management.

### Key Objectives:
1. Deploy a Windows Server 2022/2019 virtual machine with GUI Desktop Experience.
2. Deploy an Ubuntu Server 22.04 LTS virtual machine with headless CLI and OpenSSH Server.
3. Configure and balance compute and storage resources (RAM, vCPUs, dynamically allocated VDI storage, virtual network adapters).
4. Organize VM virtual disks, configuration files, and ISO images inside structured directories.
5. Validate network functionality and service accessibility.
6. Create immutable baseline snapshots for rapid disaster recovery and rollback.
7. Manage lab documentation and version control through Git and GitHub with appropriate `.gitignore` rules.

---

## 2. Host Environment & Sizing Specifications

- **Host Workstation:** Acer Nitro ANV15-41
- **Host Operating System:** Microsoft Windows 11 Home/Pro 64-bit
- **Host Processor:** AMD Ryzen Multi-Core Processor (AMD-V / SVM Virtualization enabled in BIOS)
- **Host RAM:** 16.0 GB DDR5 RAM
- **Host Storage:** High-speed NVMe PCIe SSD
- **Hypervisor:** Oracle VM VirtualBox 7.x

### Resource Budgeting:
| Resource | Host Total | Windows Server VM | Ubuntu Server VM | Remaining for Host |
| :--- | :--- | :--- | :--- | :--- |
| **RAM** | 16,384 MB (16 GB) | 4,096 MB (4 GB) | 2,048 MB (2 GB) | ~10,240 MB (10 GB) |
| **vCPU Cores** | Multi-Core Host | 2 Cores | 2 Cores | Unconstrained (Host Scheduler) |
| **Disk Max Capacity** | 512 GB+ SSD | 50.00 GB (Dynamic) | 30.00 GB (Dynamic) | Thin Provisioned (< 20 GB initial) |

---

## 3. Part 1: Windows Server VM Installation & Configuration

### VM Provisioning Parameters:
- **VM Name:** `Windows_Server`
- **Machine Type:** Microsoft Windows
- **Version:** Windows 2022 (64-bit) / Windows 2019 (64-bit)
- **Base Memory:** 4096 MB (4 GB)
- **Processor:** 2 vCPU Cores
- **Hard Disk:** VDI (VirtualBox Disk Image), Dynamically Allocated, 50.00 GB capacity
- **Storage Controller:** SATA Controller with AHCI
- **Optical Drive:** Attached official evaluation ISO (`Windows Server Evaluation Edition`)

### Installation Steps:
1. **Boot:** Power on `Windows_Server`. Press any key when prompted to boot from optical media.
2. **Localization:** Language: `English (United States)`, Time/Currency: `English (Philippines / United States)`, Keyboard: `US`.
3. **Edition Selection:** Selected **Windows Server 2022 Standard Evaluation (Desktop Experience)**.
   > *Note:* Server Core is default in some wizards, but Desktop Experience is mandatory for Server Manager GUI labs.
4. **Installation Type:** Select `Custom: Install Microsoft Server Operating System only (advanced)`.
5. **Disk Allocation:** Target `Drive 0 Unallocated Space (50.0 GB)`. Click `Next` to let Windows automatically generate EFI, MSR, and Primary NTFS partitions.
6. **Administrator Setup:**
   - Account: `Admin` or `Administrator`
   - Password: `P@ssw0rd123`
7. **Network Adapter:** VirtualBox `Settings -> Network -> Adapter 1` set to `NAT` (or `Bridged Adapter` depending on lab requirements).

### Post-Installation Verification:
- Log in by sending `Ctrl + Alt + Del` via VirtualBox Host key (`Host + Del`).
- Confirm Server Manager opens cleanly and reports all local services online.
- Open PowerShell as Administrator and run verification commands:
```powershell
# System Information Check
Get-ComputerInfo | Select-Object WindowsProductName, OsArchitecture, WindowsVersion, TotalPhysicalMemory

# Network IP Configuration
ipconfig /all

# Connectivity Test
Test-NetConnection -ComputerName 8.8.8.8 -Port 53
```

### Baseline Snapshot:
- Shut down VM gracefully: `Stop-Computer`
- In VirtualBox Manager: `Snapshots -> Take`
- **Snapshot Name:** `Clean Install – Windows Server`
- **Description:** `Pristine base OS installation of Windows Server Standard Evaluation with Desktop Experience, network verified, default Admin credentials set.`

---

## 4. Part 2: Ubuntu Server 22.04 LTS VM Installation & Configuration

### VM Provisioning Parameters:
- **VM Name:** `Ubuntu_Server`
- **Machine Type:** Linux
- **Version:** Ubuntu (64-bit)
- **Base Memory:** 2048 MB (2 GB)
- **Processor:** 2 vCPU Cores
- **Hard Disk:** VDI (VirtualBox Disk Image), Dynamically Allocated, 30.00 GB capacity
- **Storage Controller:** SATA Controller
- **Optical Drive:** Attached `ubuntu-22.04-live-server-amd64.iso`

### Installation Steps (Subiquity Installer):
1. **Boot:** Power on `Ubuntu_Server`, select `Try or Install Ubuntu Server`.
2. **Language:** English
3. **Installer Update:** Continue without updating (uses stable base installer).
4. **Keyboard Layout:** English (US)
5. **Base Installation:** Ubuntu Server (standard).
6. **Network Connections:** `enp0s3` configured via DHCP (NAT network `10.0.2.15/24`).
7. **Proxy & Archive:** Proxy left blank; confirmed archive mirror (`http://ph.archive.ubuntu.com/ubuntu/`).
8. **Guided Storage:** Selected `Use an entire disk` with `Set up this disk as an LVM group`. Ext4 filesystem mounted on `/`.
9. **Profile Configuration:**
   - Your name: `Admin`
   - Your server's name: `ubuntuserver`
   - Pick a username: `admin`
   - Choose a password: `P@ssw0rd123`
10. **SSH Setup:** Checked **[X] Install OpenSSH Server**. (Essential for remote headless administration via SSH).
11. **Featured Server Snaps:** None selected (clean baseline).
12. **Reboot:** When complete, choose `Reboot Now` and press Enter to unmount optical drive.

### Post-Installation Verification:
- Log in to the CLI console with username `admin` and password `P@ssw0rd123`.
- Execute validation commands:
```bash
# Verify IP address on interface enp0s3
ip -br a

# Check default gateway and routing table
ip route

# Verify OpenSSH daemon status
systemctl status ssh --no-pager

# Test outbound Internet connectivity
ping -c 4 8.8.8.8

# Synchronize package repositories
sudo apt update
```

### Baseline Snapshot:
- Shut down VM cleanly: `sudo poweroff`
- In VirtualBox Manager: `Snapshots -> Take`
- **Snapshot Name:** `Clean Install – Ubuntu Server`
- **Description:** `Pristine base OS installation of Ubuntu Server 22.04 LTS with OpenSSH enabled, network configured, and admin sudo account initialized.`

---

## 5. Part 3: Directory Architecture & Git Version Control

### Directory Organization:
```text
it123-labs/
├── VMs/                     # VirtualBox machine files (.vbox, .vdi) - GIT IGNORED
├── VM_ISOs/                 # Downloaded OS installation ISO images - GIT IGNORED
└── Week2/                   # Lab deliverables and documentation
    ├── lab_notes.md         # Technical lab notes
    ├── screenshots/         # Verified screen captures
    ├── Lab Report 1.docx    # Formatted Word report
    └── Lab Report 1.pdf     # Submitted LMS PDF document
```

### .gitignore Configuration:
To prevent committing multi-gigabyte virtual disks (`.vdi`), snapshot files (`.sav`), or ISOs to GitHub:
```gitignore
# Virtual Machine Disks and Files
*.vdi
*.vbox
*.vbox-prev
*.sav
*.nvram
*.vmdk
*.vhdx

# ISO Images
*.iso
*.img

# Logs and Caches
*.log
Logs/
VMs/
VM_ISOs/
```

### Git Workflow Commands:
```bash
# Check status of tracked and untracked files
git status

# Stage lab documentation and notes
git add .

# Commit changes
git commit -m "Week 2 – Windows & Ubuntu Server installation"

# Push to GitHub remote main branch
git push origin main
```

---

## 6. Summary of VM Configurations

| Parameter | Windows Server VM | Ubuntu Server VM |
| :--- | :--- | :--- |
| **VM Name** | `Windows_Server` | `Ubuntu_Server` |
| **Operating System** | Windows Server 2022/2019 Standard Evaluation | Ubuntu Server 22.04 LTS |
| **Environment** | Desktop Experience (GUI) | Headless Server (CLI) |
| **Allocated RAM** | 4096 MB (4.0 GB) | 2048 MB (2.0 GB) |
| **vCPU Cores** | 2 vCPUs | 2 vCPUs |
| **Storage Capacity** | 50.00 GB | 30.00 GB |
| **Virtual Disk Type** | VDI (Dynamically Allocated) | VDI (Dynamically Allocated) |
| **Filesystem** | NTFS (GPT) | ext4 on LVM |
| **Network Mode** | NAT / Bridged Adapter | NAT / Bridged Adapter |
| **Default Username** | `Admin` / `Administrator` | `admin` (sudo privileged) |
| **Default Password** | `P@ssw0rd123` | `P@ssw0rd123` |
| **Remote Access** | Server Manager / WinRM / RDP | OpenSSH Server (Port 22) |
| **Baseline Snapshot** | `Clean Install – Windows Server` | `Clean Install – Ubuntu Server` |

---

## 7. Required Screenshots Checklist

1. **`01_windows_server_vm_settings.png`**: VirtualBox Manager settings screen for `Windows_Server` showing 4096 MB RAM, 2 vCPUs, 50 GB VDI, and Network Adapter.
2. **`02_windows_server_login.png`**: Successful desktop login showing Server Manager Dashboard and active local server metrics.
3. **`03_ubuntu_server_vm_settings.png`**: VirtualBox Manager settings screen for `Ubuntu_Server` showing 2048 MB RAM, 2 vCPUs, 30 GB VDI, and Network Adapter.
4. **`04_ubuntu_server_login.png`**: Console terminal showing successful login as `admin`, output of `systemctl status ssh`, and `ip a`.
5. **`05_vm_snapshots.png`**: VirtualBox Snapshots window showing `Clean Install – Windows Server` and `Clean Install – Ubuntu Server`.

---

## 8. Technical Problem-Solving & Observations

1. **Hardware Virtualization (AMD-V):** Verified SVM mode enabled in Acer Nitro BIOS. Without hardware virtualization, 64-bit guest operating systems cannot launch inside VirtualBox.
2. **Dynamic vs. Fixed Storage:** Dynamically allocated VDI prevented reserving 80 GB of physical SSD space upfront, only consuming the actual ~14 GB (Windows) and ~3.8 GB (Ubuntu) required for initial installation.
3. **NAT vs. Bridged Network Selection:** NAT mode was chosen for initial deployment to guarantee reliable outbound internet connectivity for updates (`apt update` / Windows Update) while avoiding campus Wi-Fi MAC restrictions.
4. **Snapshot Hygiene:** Baseline snapshots captured immediately after post-install verification ensure safe experimentation in upcoming lab modules.
