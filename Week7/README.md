# IT 123 – Week 7: Configuring and Securing Network Services
**Lab Performance 2 (Fourth Graded Assessment)**

- **Student:** Jhon Kenneth S. Aguilar
- **Program & Section:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui, Albay
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## Deliverables Summary

This repository directory contains the complete implementation, configuration files, terminal verification outputs, and official documentation for **Lab Performance 2**.

### 📁 Files in this Directory:

1. **Official Reports:**
   - [`IT123B_LabPerf2_Aguilar.docx`](IT123B_LabPerf2_Aguilar.docx) – Official Bicol University Polangui document format with complete lab sections, verification tables, screenshots, and technical reflections.
   - [`IT123B_LabPerf2_Aguilar.pdf`](IT123B_LabPerf2_Aguilar.pdf) – High-resolution exported PDF document.

2. **Laboratory Documentation:**
   - [`week7_notes.md`](week7_notes.md) – Complete Markdown notes detailing commands, system configurations, and verification steps.

3. **Service Configuration Files (`configs/`):**
   - [`configs/named.conf.local`](configs/named.conf.local) – BIND9 local zone configuration (`example.local` & `company.local`).
   - [`configs/db.example.local`](configs/db.example.local) – Part 1 forward zone database.
   - [`configs/db.company.local`](configs/db.company.local) – Part 6 student exercise zone database.
   - [`configs/dhcpd.conf`](configs/dhcpd.conf) – ISC DHCP server configuration (`192.168.1.160` – `192.168.1.170`).
   - [`configs/company.local.conf`](configs/company.local.conf) – Apache VirtualHost configuration.
   - [`configs/index.html`](configs/index.html) – Corporate intranet landing page displaying *"Welcome to Company Intranet Server IP: 192.168.1.150"*.
   - [`configs/vsftpd.conf`](configs/vsftpd.conf) – Hardened vsftpd FTP server configuration.
   - [`configs/ufw.rules.txt`](configs/ufw.rules.txt) – Active UFW firewall rule specifications.

4. **Terminal Verification Evidence (`screenshots/`):**
   - [`screenshots/01_dns_bind9_verification.png`](screenshots/01_dns_bind9_verification.png) – BIND9 zone syntax verification & `nslookup` resolution.
   - [`screenshots/02_dhcp_server_status.png`](screenshots/02_dhcp_server_status.png) – ISC DHCP server syntax test & active UDP 67 socket.
   - [`screenshots/03_apache_http_verification.png`](screenshots/03_apache_http_verification.png) – Apache2 HTTP server active status & `curl -I` HTTP 200 OK header.
   - [`screenshots/04_vsftpd_ftp_verification.png`](screenshots/04_vsftpd_ftp_verification.png) – vsftpd active daemon & authenticated FTP login.
   - [`screenshots/05_ufw_firewall_status.png`](screenshots/05_ufw_firewall_status.png) – UFW status verbose & active port access control matrix.
   - [`screenshots/06_student_exercise_company_local.png`](screenshots/06_student_exercise_company_local.png) – `company.local` DNS query resolution & Intranet landing page verification.
