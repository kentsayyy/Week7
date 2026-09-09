# IT 123 – System Administration
## Week 4 Laboratory Notes: File Systems and Storage Management

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course & Year:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## 1. Laboratory Overview & Objectives

This laboratory investigates the fundamentals of storage engineering, disk partitioning, filesystem formatting (NTFS, FAT32, ext4), volume mounting, and discretionary access control.

### Key Objectives:
1. Identify and compare filesystem architectures (NTFS vs. FAT32 vs. ext4).
2. Create partitions and format volumes with appropriate filesystems.
3. Mount storage volumes in Linux and assign drive letters in Windows.
4. Apply and test folder permissions on newly provisioned storage.
5. Solve the University Project Storage scenario with segregated access for Faculty and Students.

---

## 2. Command Reference & Implementation Guide

### Part 1 & Part 4: Windows Server Storage Operations

In Windows Server (PowerShell as Administrator):

#### Option A: Partition Existing Unallocated Space or New Disk
```powershell
# 1. Inspect Disks
Get-Disk

# 2. Partition and Format Drive E: (NTFS, 5 GB for FacultyDocs)
New-Partition -DiskNumber 0 -Size 5GB -DriveLetter E | Format-Volume -FileSystem NTFS -NewFileSystemLabel "FacultyDocs" -Confirm:$false

# 3. Partition and Format Drive F: (FAT32, 5 GB for Project Exercise)
New-Partition -DiskNumber 0 -Size 5GB -DriveLetter F | Format-Volume -FileSystem FAT32 -NewFileSystemLabel "Projects" -Confirm:$false

# 4. Create Folders
New-Item -Path "E:\FacultyDocs", "F:\SharedFaculty", "F:\SharedStudents" -ItemType Directory -Force

# 5. Apply NTFS Permissions on E:\FacultyDocs
icacls "E:\FacultyDocs" /inheritance:d
icacls "E:\FacultyDocs" /remove "Users"
icacls "E:\FacultyDocs" /grant "FacultyGroup:(OI)(CI)M"
icacls "E:\FacultyDocs" /grant "StudentGroup:(OI)(CI)RX"

# 6. Verify Volumes and Permissions
Get-Volume E, F
icacls "E:\FacultyDocs"
```

---

### Part 2 & Part 4: Ubuntu Server Storage Operations

In Ubuntu Server terminal:

#### 1. Add Virtual Hard Disk in VirtualBox:
- In VirtualBox Manager -> `Ubuntu_Server` Settings -> `Storage` -> SATA Controller -> click `Add Hard Disk` -> Create a 5 GB or 10 GB VDI.
- Or if running live without reboot, you can create a test block device using a loop disk:
```bash
# Optional loop device if virtual disk is not yet attached:
sudo truncate -s 10G /disk.img
sudo losetup -fP /disk.img
```

#### 2. Partition and Format with ext4:
```bash
# Identify disk (/dev/sdb or /dev/loop0)
lsblk

# Partition disk with fdisk (create 1 primary partition)
echo -e "n\np\n1\n\n\nw" | sudo fdisk /dev/sdb

# Format with ext4 filesystem
sudo mkfs.ext4 -F /dev/sdb1

# Create mount points
sudo mkdir -p /mnt/data /projectdata

# Mount partition
sudo mount /dev/sdb1 /mnt/data

# Verify mount
df -hT /mnt/data
```

#### 3. Assign Permissions:
```bash
# /mnt/data -> Only root and facultygrp (mode 770)
sudo chown root:facultygrp /mnt/data
sudo chmod 770 /mnt/data

# Verify
ls -ld /mnt/data
```

#### 4. Access Verification:
```bash
# Test as faculty2 (Success)
su - faculty2 -c "touch /mnt/data/faculty_doc.txt"

# Test as student4 (Denied)
su - student4 -c "touch /mnt/data/student_doc.txt"
```

---

## 3. Reflection Questions & Answers

### 1. What challenges did you encounter in creating partitions and managing permissions?
*Answer:*  
The primary challenge encountered was understanding the architectural limitations of legacy filesystems such as FAT32 compared to modern journaling filesystems like NTFS and ext4. Formatting a volume as FAT32 on Windows revealed that FAT32 does not support native file-level Access Control Lists (ACLs), making it impossible to assign granular permissions directly to folders without network-share restrictions. Furthermore, in Linux, ensuring persistent mounting across reboots requires correctly configuring `/etc/fstab` using filesystem UUIDs instead of volatile device names (`/dev/sdb1`), preventing boot failures if drive enumeration changes.

### 2. How can these skills be applied in real-world system administration duties?
*Answer:*  
Storage provisioning and access control are critical daily responsibilities in enterprise infrastructure engineering. In real-world environments, administrators isolate operating system volumes from database, application, and user storage to prevent user data overconsumption from exhausting root disk capacity and causing catastrophic server crashes. Additionally, enforcing role-based discretionary permissions ensures strict adherence to data governance policies and compliance with regulatory frameworks (such as the Data Privacy Act of 2012 / RA 10173 in the Philippines), guaranteeing that sensitive financial, faculty, and student records remain strictly confidential and protected from unauthorized modification.
