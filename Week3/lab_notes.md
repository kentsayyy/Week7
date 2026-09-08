# IT 123 – System Administration
## Week 3 Laboratory Notes: User and Group Management in Windows & Linux

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course & Year:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## 1. Laboratory Overview & Objectives

This laboratory covers the configuration and management of local users, functional security groups, and filesystem permissions across both Windows Server and Ubuntu Linux Server.

### Key Objectives:
1. Create, modify, and delete local user accounts on Windows and Linux.
2. Formulate security groups and assign user principals based on Role-Based Access Control (RBAC).
3. Enforce and test NTFS Access Control Lists (ACLs) in Windows Server.
4. Enforce and test POSIX octal permission modes (770, 750) in Ubuntu Server.
5. Execute the University Laboratory Exercise partitioning Faculty and Student storage resources.
6. Conduct empirical verification testing recording both successful access and explicit permission denials.

---

## 2. Quick Command Reference

### Windows Server (PowerShell Run as Administrator)

#### Part 1: Initial User, Group & Folder
```powershell
# Create user student1 with password User@123
$Pass = ConvertTo-SecureString "User@123" -AsPlainText -Force
New-LocalUser -Name "student1" -Password $Pass -Description "Test account for Week 3 Lab" -ChangePasswordAtLogon $true

# Create group LabUsers and add student1
New-LocalGroup -Name "LabUsers" -Description "General Lab Users Group"
Add-LocalGroupMember -Group "LabUsers" -Member "student1"

# Create folder C:\LabData and grant Read & Execute to LabUsers
New-Item -Path "C:\LabData" -ItemType Directory -Force
icacls "C:\LabData" /grant "LabUsers:(OI)(CI)RX" /t
```

#### Part 4: University Laboratory Exercise
```powershell
# Create users: faculty1, student3
$Pass = ConvertTo-SecureString "User@123" -AsPlainText -Force
New-LocalUser -Name "faculty1" -Password $Pass -Description "University Faculty Member 1"
New-LocalUser -Name "student3" -Password $Pass -Description "University Student Account 3"

# Create groups: FacultyGroup, StudentGroup
New-LocalGroup -Name "FacultyGroup"
New-LocalGroup -Name "StudentGroup"

# Assign users to groups
Add-LocalGroupMember -Group "FacultyGroup" -Member "faculty1"
Add-LocalGroupMember -Group "StudentGroup" -Member "student3"

# Create folders
New-Item -Path "C:\FacultyData", "C:\StudentData" -ItemType Directory -Force

# Permissions for C:\FacultyData (Only FacultyGroup has Modify rights)
icacls "C:\FacultyData" /inheritance:d
icacls "C:\FacultyData" /remove "Users"
icacls "C:\FacultyData" /grant "FacultyGroup:(OI)(CI)M"

# Permissions for C:\StudentData (Only StudentGroup has Read rights)
icacls "C:\StudentData" /inheritance:d
icacls "C:\StudentData" /remove "Users"
icacls "C:\StudentData" /grant "StudentGroup:(OI)(CI)RX"
icacls "C:\StudentData" /grant "FacultyGroup:(OI)(CI)M"
```

---

### Ubuntu Server (Terminal / Bash)

#### Part 2: Initial User, Group & Directory
```bash
# Create student2 with password User@123
sudo adduser student2
# (Enter password User@123 and press Enter for details)

# Modify description
sudo usermod -c "Test account for Week 3 Lab" student2

# Create group labusers and add student2
sudo groupadd labusers
sudo usermod -aG labusers student2

# Create directory /labdata with mode 770
sudo mkdir /labdata
sudo chown root:labusers /labdata
sudo chmod 770 /labdata
```

#### Part 4: University Laboratory Exercise
```bash
# Create groups: facultygrp, studentgrp
sudo groupadd facultygrp
sudo groupadd studentgrp

# Create users: faculty2, student4 with password User@123
sudo useradd -m -s /bin/bash -g facultygrp -c "University Faculty 2" faculty2
echo "faculty2:User@123" | sudo chpasswd

sudo useradd -m -s /bin/bash -g studentgrp -c "University Student 4" student4
echo "student4:User@123" | sudo chpasswd

# Create directories
sudo mkdir /facultydata /studentdata

# Permissions for /facultydata (Only facultygrp has read/write)
sudo chown root:facultygrp /facultydata
sudo chmod 770 /facultydata

# Permissions for /studentdata (Only studentgrp has read-only)
sudo chown root:studentgrp /studentdata
sudo chmod 750 /studentdata
```

---

## 3. Verification Test Results Matrix

| User | Target Resource | Action Tested | Expected Result | Actual Result |
| :--- | :--- | :--- | :--- | :--- |
| `student1` | `C:\LabData` | Read files | Success | ALLOWED |
| `student1` | `C:\LabData` | Create/modify files | Access Denied | DENIED (0x80070005) |
| `faculty1` | `C:\FacultyData` | Create/edit files | Success (Modify) | ALLOWED |
| `student3` | `C:\FacultyData` | Access directory | Access Denied | DENIED |
| `student3` | `C:\StudentData` | Read files | Success (Read-Only) | ALLOWED |
| `student3` | `C:\StudentData` | Create/edit files | Access Denied | DENIED |
| `student2` | `/labdata` (770) | `touch file1.txt` | Success | ALLOWED |
| `student2` | `/labdata` (750) | `touch file2.txt` | Permission Denied | DENIED |
| `faculty2` | `/facultydata` (770) | `touch syllabus.pdf` | Success | ALLOWED |
| `student4` | `/facultydata` (770) | `cd /facultydata` | Permission Denied | DENIED |
| `student4` | `/studentdata` (750) | `ls /studentdata` | Success (Read) | ALLOWED |
| `student4` | `/studentdata` (750) | `touch grades.txt` | Permission Denied | DENIED |
