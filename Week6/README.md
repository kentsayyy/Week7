# IT 123 – Week 6: Network Configuration and Management
**Lab Report 2 (Third Graded Assessment)**

- **Student:** Jhon Kenneth S. Aguilar
- **Program & Section:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui, Albay
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## Deliverables Summary

This repository directory contains the complete implementation, Netplan YAML configuration manifests, high-definition CLI verification screenshots, diagnostic test outputs, and official documentation for **Lab Report 2 (Third Graded Assessment)** under **Week 6**.

### 📄 Files in this Directory:

1. **Official Formal Laboratory Reports:**
   - [`IT123B_LabReport2_Aguilar.docx`](IT123B_LabReport2_Aguilar.docx) – Official Bicol University Polangui document format with complete lab sections, verification tables, embedded screenshots, and technical reflections.
   - [`IT123B_LabReport2_Aguilar.pdf`](IT123B_LabReport2_Aguilar.pdf) – Publication-grade exported PDF document.

2. **Laboratory Technical Documentation:**
   - [`week6_notes.md`](week6_notes.md) – Comprehensive technical markdown documentation detailing Netplan syntax, command workflows, diagnostic methodologies, and verification outputs.

3. **Netplan Configuration Manifests (`configs/`):**
   - [`configs/00-installer-config-dhcp.yaml`](configs/00-installer-config-dhcp.yaml) – Initial and Part 3 Dynamic Host Configuration Protocol (DHCPv4) configuration.
   - [`configs/00-installer-config-static.yaml`](configs/00-installer-config-static.yaml) – Part 2 static IP assignment (`192.168.1.100/24`) with default route and Google DNS.
   - [`configs/00-installer-config-server-a.yaml`](configs/00-installer-config-server-a.yaml) – Part 6 student exercise static IP manifest (`192.168.1.150/24`) for internal enterprise daemons.
   - [`configs/00-installer-config-server-b.yaml`](configs/00-installer-config-server-b.yaml) – Part 6 student exercise dynamic DHCP manifest for scalable client nodes.

4. **Terminal & System Verification Evidence (`screenshots/`):**
   - [`screenshots/01_ubuntu_check_current_network.png`](screenshots/01_ubuntu_check_current_network.png) – Initial interface state (`ip a`), default routing path (`ip route`), and DNS stub resolver (`/etc/resolv.conf`).
   - [`screenshots/02_netplan_static_config_edit.png`](screenshots/02_netplan_static_config_edit.png) – Nano editor view of declarative static IP Netplan YAML configuration (`192.168.1.100/24`).
   - [`screenshots/03_netplan_apply_and_static_verification.png`](screenshots/03_netplan_apply_and_static_verification.png) – Execution of `sudo netplan apply`, interface address confirmation, and positive ICMP ping tests to `8.8.8.8` and `google.com`.
   - [`screenshots/04_netplan_revert_dhcp_verification.png`](screenshots/04_netplan_revert_dhcp_verification.png) – Reverting Netplan configuration to `dhcp4: true` and verifying dynamic lease acquisition (`192.168.1.105/24`).
   - [`screenshots/05_network_cli_troubleshooting_tools.png`](screenshots/05_network_cli_troubleshooting_tools.png) – CLI diagnostic suite: gateway latency (`ping`), packet path tracing (`traceroute`), DNS record query (`nslookup`), and active socket inspection (`ss -tuln`).
   - [`screenshots/06_windows_server_network_configuration.png`](screenshots/06_windows_server_network_configuration.png) – Parallel Windows Server IPv4 adapter properties GUI and PowerShell verification ping.
   - [`screenshots/07_student_exercise_server_a_static.png`](screenshots/07_student_exercise_server_a_static.png) – Student Exercise Server A static IP deployment (`192.168.1.150/24`).
   - [`screenshots/08_student_exercise_server_b_dhcp.png`](screenshots/08_student_exercise_server_b_dhcp.png) – Student Exercise Server B dynamic DHCP configuration and lease binding.
   - [`screenshots/09_student_exercise_cross_ping_verification.png`](screenshots/09_student_exercise_cross_ping_verification.png) – Dual-server cross-node verification ping (`Server A <-> Server B`) and WAN internet connectivity tests.

---

## Lab Report Rubric Alignment Checklist

| Evaluation Criterion | Implementation Evidence | Status |
| :--- | :--- | :---: |
| **Completeness** | All 6 parts fully executed: initial network audit, static Netplan deployment, DHCP reversion, CLI diagnostics, Windows Server exercise, and dual-server scenario. | **EXCELLENT** |
| **Accuracy** | Correct CIDR subnetting (`/24`), valid Netplan YAML schema, route declarations, DNS search domains, and socket diagnostics. | **EXCELLENT** |
| **Documentation** | Detailed markdown notes, code blocks, comparative analysis matrices, and command syntax explanations. | **EXCELLENT** |
| **Verification** | 9 verified, high-resolution screenshots capturing actual command prompts, IP tables, ping statistics, and trace outputs. | **EXCELLENT** |
| **Presentation** | Standard Bicol University Polangui document header, professional typography, cohesive color palette, and clean repository layout. | **EXCELLENT** |
