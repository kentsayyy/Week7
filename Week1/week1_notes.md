# IT 123 – System Administration
## Week 1 Laboratory Notes: Preliminary Setup – Installing Essential Tools

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course & Year:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui, Albay
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## 1. Laboratory Overview & Objectives

System administration requires a reliable, secure, and standardized local administrative workstation capable of running virtualized enterprise server operating systems, writing infrastructure automation scripts, inspecting configuration files, and managing version-controlled laboratory outputs.

The primary objectives of this preliminary laboratory session include:
1. Installing and validating the three primary course tools: **Visual Studio Code**, **Git Distributed Version Control**, and **Oracle VM VirtualBox**.
2. Verifying hardware and operating system virtualization capabilities (Intel VT-x / AMD-V enabled in UEFI/BIOS).
3. Establishing the canonical repository structure for all laboratory outputs under `it123-labs/`.
4. Configuring global Git identity and verifying remote repository synchronizations to GitHub.
5. Executing an end-to-end setup verification test via `setup_test.txt`.

---

## 2. Hardware Prerequisites & Workstation Audit

| Specification Component | Recommended Requirement | Host Workstation Verified State | Compliance Status |
| :--- | :--- | :--- | :---: |
| **Processor (CPU)** | 64-bit multi-core with hardware virtualization | AMD Ryzen / Intel Core 64-bit Architecture | **VERIFIED** |
| **Hardware Virtualization** | AMD-V or Intel VT-x enabled in BIOS/UEFI | Hypervisor & Virtualization Extensions Active | **VERIFIED** |
| **Random Access Memory (RAM)** | Minimum 8 GB (Recommended 16 GB+) | 16 GB DDR4/DDR5 Dual-Channel | **VERIFIED** |
| **Available Disk Storage** | > 50 GB free NVMe/SSD storage for VM VDIs | > 100 GB High-Speed Solid State Drive | **VERIFIED** |
| **Host Operating System** | Windows 10/11 64-bit / macOS / Linux | Windows 11 64-bit Professional | **VERIFIED** |

---

## 3. Part 1: Installing and Configuring Visual Studio Code (VS Code)

Visual Studio Code serves as the primary Integrated Development Environment (IDE) for editing Bash shell scripts, PowerShell maintenance routines, Netplan YAML manifests, BIND DNS zone records, and Markdown technical reports.

### Installation & Verification Steps:
1. Downloaded the Windows x64 User Installer (`VSCodeUserSetup-x64.exe`) from the official repository (`https://code.visualstudio.com/`).
2. Checked **"Add to PATH"** to ensure direct CLI invocation from Command Prompt and PowerShell.
3. Enabled Windows Explorer context menu options (*"Open with Code"*).
4. Verified successful installation via integrated terminal:
   ```powershell
   code --version
   # Expected Output:
   # 1.127.0
   # 4fe60c8b1cdac1c4c174f2fb180d0d758272d713
   # x64
   ```

---

## 4. Part 2: Installing and Configuring Git Distributed Version Control

Git provides distributed tracking, immutable commit history, and branching workflows for all laboratory configurations, scripts, and documentation.

### Installation & Identity Configuration:
1. Downloaded Git for Windows (`Git-64-bit.exe`) from `https://git-scm.com/downloads`.
2. Selected default options: *"Git from the command line and also from 3rd-party software"* and OpenSSH client.
3. Executed version verification and configured global identity:
   ```powershell
   # 1. Verify Git binary
   git --version
   # Output: git version 2.52.0.windows.1

   # 2. Configure Global User Identity
   git config --global user.name "kenneth"
   git config --global user.email "kennethaguilar436@gmail.com"

   # 3. Configure Default Branch Name & Line Ending Normalization
   git config --global init.defaultBranch main
   git config --global core.autocrlf true

   # 4. Audit Configured Parameters
   git config --list --show-origin
   ```

---

## 5. Part 3: Installing Oracle VM VirtualBox (Type-2 Hypervisor)

Oracle VM VirtualBox acts as the Type-2 hypervisor hosting guest virtual machines throughout the semester, including Ubuntu Server 22.04 LTS and Windows Server 2022.

### Installation & Virtualization Verification:
1. Downloaded the Oracle VM VirtualBox 7.2.x Windows installer and Extension Pack from `https://www.virtualbox.org/wiki/Downloads`.
2. Verified that hardware-assisted virtualization (VT-x/AMD-V) is active and accessible to hypervisor processes.
3. Checked binary status via CLI and GUI:
   ```powershell
   & "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
   # Output: 7.2.16r174877
   ```
4. Launched Oracle VM VirtualBox Manager and confirmed host readiness.

---

## 6. Part 4 & 5: Repository Organization & Setup Testing

### Canonical Directory Structure:
```
it123-labs/
├── Week1/         # Preliminary Setup: VS Code, Git, VirtualBox
├── Week2/         # Virtual Machine Provisioning & OS Installation
├── Week3/         # User and Group Management (Windows & Linux)
├── Week4/         # File Systems and Storage Management
├── Week5/         # System Initialization and Service Management
├── Week6/         # Network Configuration and Management
├── Week7/         # Configuring and Securing Network Services
└── ...
```

### Verification Test Execution:
```powershell
# 1. Navigate to Week1 directory
cd "it123-labs\Week1"

# 2. Generate verification file
echo "Week 1 Setup Complete" > setup_test.txt

# 3. Stage, commit, and push to GitHub repository
git add Week1/
git commit -m "Week 1 setup complete: VS Code, Git, VirtualBox & workspace structure"
git push origin main
```

### Verification Findings:
- File `setup_test.txt` successfully written and validated.
- Git commit created cleanly without merge conflicts.
- Remote repository synchronization verified on GitHub.
