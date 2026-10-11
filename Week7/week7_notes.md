# IT 123 – System Administration
## Week 7 Laboratory Guide & Performance Documentation #2: Configuring and Securing Network Services

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course, Year & Section:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui
- **Academic Term:** 1st Semester, A.Y. 2026–2027
- **Deliverable:** Lab Performance 2 (Fourth Graded Assessment)

---

## 1. Laboratory Overview & Objectives

In modern operating systems engineering and enterprise network administration, services constitute the application-layer infrastructure that enables host discovery, automated addressing, name resolution, web hosting, and controlled file exchange.

### Key Objectives:
1. **DNS Architecture (BIND9):** Configure authoritative forward lookup zones (`example.local` and `company.local`), SOA records, NS records, and A records. Verify query resolution using `named-checkzone`, `nslookup`, and `dig`.
2. **DHCP Implementation (ISC DHCP Server):** Define subnet addressing pools (`192.168.1.160` – `192.168.1.170`), default gateways, lease times, and automatic internal DNS distribution for client endpoints.
3. **HTTP Web Server Deployment (Apache2):** Provision Apache HTTP Server, verify daemon state, HTTP status codes, response headers, and deploy the custom company intranet portal.
4. **FTP Protocol Deployment & Containment (vsftpd):** Configure Very Secure FTP Daemon with authenticated PAM access, write capabilities, chroot jail containment, and restriction to the private LAN interface.
5. **Security Hardening & Packet Filtering (UFW):** Establish a default-deny ingress firewall policy, open minimal application ports (TCP 22, 53, 80, 21, 40000–40100, and UDP 53), and inspect active rules via `ufw status verbose`.
6. **Small Office Network Scenario Integration:** Fulfill the complete corporate multi-service deployment scenario connecting DNS, DHCP, Web, and FTP under a unified local topology.

---

## 2. Server Environment & Network Configuration

| Parameter | Configuration Value | Description |
| :--- | :--- | :--- |
| **Operating System** | Ubuntu Server 24.04 LTS (x86_64) | Base Server Platform |
| **Server Hostname** | `ubuntuserver` (`ns1.company.local`) | Core Infrastructure Daemon Host |
| **Static IP Address** | `192.168.1.150 /24` | Configured from Week 6 static assignment |
| **Subnet Mask** | `255.255.255.0` | Class C private network boundary |
| **Default Gateway** | `192.168.1.1` | Local border gateway / upstream router |
| **Dynamic DHCP Pool** | `192.168.1.160` – `192.168.1.170` | 11 assignable client host addresses |
| **Authoritative Domain** | `company.local` / `example.local` | Private enterprise namespace |
| **Firewall Profile** | UFW (Uncomplicated Firewall) | Stateful packet filtering (Default Deny Inbound) |

---

## 3. Step-by-Step Implementation Guide

### Part 1 – DNS Configuration (BIND9)

#### 1. Package Installation
```bash
sudo apt update
sudo apt install bind9 bind9utils -y
```

#### 2. Configure Local Zone Declaration
Edit `/etc/bind/named.conf.local`:
```bash
sudo nano /etc/bind/named.conf.local
```
Add the authoritative zone definition:
```bind
zone "example.local" {
    type master;
    file "/etc/bind/db.example.local";
};
```

#### 3. Create Zone Database File
Copy template from `/etc/bind/db.local`:
```bash
sudo cp /etc/bind/db.local /etc/bind/db.example.local
sudo nano /etc/bind/db.example.local
```
Configure records:
```bind
;
; BIND data file for example.local zone
;
$TTL    604800
@       IN      SOA     ns1.example.local. admin.example.local. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.example.local.
ns1     IN      A       192.168.1.150
www     IN      A       192.168.1.150
```

#### 4. Syntax Audit & Service Startup
```bash
# Verify configuration and zone syntax
sudo named-checkconf
sudo named-checkzone example.local /etc/bind/db.example.local

# Restart and inspect service status
sudo systemctl restart bind9
sudo systemctl status bind9

# Perform query resolution tests
nslookup www.example.local 127.0.0.1
dig @127.0.0.1 www.example.local +short
```

---

### Part 2 – DHCP Configuration (ISC-DHCP-Server)

#### 1. Package Installation
```bash
sudo apt install isc-dhcp-server -y
```

#### 2. Bind Interface
In `/etc/default/isc-dhcp-server`:
```bash
INTERFACESv4="enp0s3"
```

#### 3. Configure Pool & Leases in `/etc/dhcp/dhcpd.conf`
```bash
sudo nano /etc/dhcp/dhcpd.conf
```
Add subnet block:
```text
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.160 192.168.1.170;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    default-lease-time 600;
    max-lease-time 7200;
}
```

#### 4. Syntax Audit & Daemon Startup
```bash
# Validate syntax
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# Restart and enable service
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server

# Verify listening socket
ss -uln | grep 67
```

---

### Part 3 – HTTP Service Configuration (Apache2)

#### 1. Package Installation & Service Verification
```bash
# Install Apache2
sudo apt install apache2 -y

# Restart and enable service
sudo systemctl restart apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

#### 2. Functional Verification
```bash
# Probe HTTP response headers
curl -I http://192.168.1.150

# Verify Apache default landing page
curl -s http://192.168.1.150 | grep -i "<title>"
```
*Result: `HTTP/1.1 200 OK` confirms active web server responding on port 80.*

---

### Part 4 – FTP Service Configuration (vsftpd)

#### 1. Package Installation
```bash
sudo apt install vsftpd -y
```

#### 2. Enable Write Permissions
Edit `/etc/vsftpd.conf`:
```bash
sudo nano /etc/vsftpd.conf
```
Ensure:
```ini
write_enable=YES
local_enable=YES
anonymous_enable=NO
```

#### 3. Service Restart & Connectivity Test
```bash
sudo systemctl restart vsftpd
sudo systemctl status vsftpd

# Test interactive FTP connection
ftp 192.168.1.150
```

---

### Part 5 – Securing Services & Firewall Hardening

#### 1. Restrict FTP to Local Network Interface
In `/etc/vsftpd.conf`:
```ini
listen_address=192.168.1.150
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
pasv_address=192.168.1.150
```
Restart vsftpd:
```bash
sudo systemctl restart vsftpd
```

#### 2. Stateful Packet Filtering with UFW
```bash
# Establish default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Open required application ports
sudo ufw allow 22/tcp comment 'OpenSSH Remote Management'
sudo ufw allow 53/tcp comment 'BIND9 DNS TCP'
sudo ufw allow 53/udp comment 'BIND9 DNS UDP'
sudo ufw allow 80/tcp comment 'Apache HTTP Web Server'
sudo ufw allow 21/tcp comment 'vsftpd FTP Command Channel'
sudo ufw allow 40000:40100/tcp comment 'vsftpd Passive Data Transfer'

# Enable firewall and inspect rule table
sudo ufw enable
sudo ufw status verbose
```

---

### Part 6 – Student Exercise: Enterprise Small Office Network Scenario

#### Scenario Requirements:
1. Dynamic host addressing for employees (`192.168.1.160` – `192.168.1.170`).
2. Company website hosted internally on `company.local` resolving `www.company.local` to server IP (`192.168.1.150`).
3. Apache configured to display custom intranet page:
   `Welcome to Company Intranet Server IP: 192.168.1.150`
4. Hardened FTP file sharing restricted strictly to LAN users.

#### Implementation Steps:

##### 1. Authoritative DNS Zone for `company.local`
In `/etc/bind/named.conf.local`:
```bind
zone "company.local" {
    type master;
    file "/etc/bind/db.company.local";
};
```

Create `/etc/bind/db.company.local`:
```bind
; /etc/bind/db.company.local
$TTL    604800
@       IN      SOA     ns1.company.local. admin.company.local. (
                     2026101101         ; Serial Number
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.company.local.
@       IN      A       192.168.1.150
ns1     IN      A       192.168.1.150
www     IN      A       192.168.1.150
ftp     IN      A       192.168.1.150
```

Verify and restart:
```bash
sudo named-checkzone company.local /etc/bind/db.company.local
sudo systemctl restart bind9
nslookup www.company.local 127.0.0.1
```

##### 2. DHCP Scope for Small Office
In `/etc/dhcp/dhcpd.conf`:
```text
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.160 192.168.1.170;
    option routers 192.168.1.1;
    option domain-name-servers 192.168.1.150, 8.8.8.8;
    option domain-name "company.local";
    default-lease-time 600;
    max-lease-time 7200;
}
```
Restart DHCP service:
```bash
sudo systemctl restart isc-dhcp-server
```

##### 3. Apache Corporate Intranet Web Page
Update `/var/www/html/index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Company Intranet</title>
</head>
<body style="font-family: Arial, sans-serif; text-align: center; margin-top: 50px;">
    <h1>Welcome to Company Intranet</h1>
    <p>Server IP: 192.168.1.150</p>
    <p>IT Infrastructure Services: DNS | DHCP | Apache | Secure FTP</p>
</body>
</html>
```

Test retrieval:
```bash
curl http://www.company.local
```

##### 4. Secure LAN FTP Verification
```bash
# Verify listen address binding
grep listen_address /etc/vsftpd.conf
# Output: listen_address=192.168.1.150

# Test authentication
ftp 192.168.1.150
```

---

## 4. Verification Evidence & Output Logs

| Figure | Image File | Description & Verification Evidence |
| :--- | :--- | :--- |
| **Figure 1** | `screenshots/01_dns_bind9_verification.png` | BIND9 zone syntax validation (`named-checkzone example.local` -> OK) and `nslookup www.example.local 127.0.0.1` resolving to `192.168.1.150`. |
| **Figure 2** | `screenshots/02_dhcp_server_status.png` | ISC DHCP Server syntax check success (`dhcpd -t`), active running daemon status, and UDP port 67 socket confirmation. |
| **Figure 3** | `screenshots/03_apache_http_verification.png` | Apache2 active status with 55 worker threads, `curl -I` returning `HTTP/1.1 200 OK` header, and title verification. |
| **Figure 4** | `screenshots/04_vsftpd_ftp_verification.png` | vsftpd active daemon, interactive login session showing `220 (vsFTPd 3.0.5)` banner and `230 Login successful` PAM authentication. |
| **Figure 5** | `screenshots/05_ufw_firewall_status.png` | UFW status verbose verifying active state, default incoming deny, and explicit ingress rules for ports 22, 53, 80, 21, and 40000–40100. |
| **Figure 6** | `screenshots/06_student_exercise_company_local.png` | Authoritative `company.local` zone resolution via `nslookup` and `curl http://www.company.local` displaying required intranet header and Server IP: 192.168.1.150. |

---

## 5. Technical Reflection & System Administration Insights

### 1. Interdependence of DNS, DHCP, and HTTP
In modern IT infrastructure, these services constitute an integrated operational pipeline. DHCP automates client network admission and configures the client DNS resolver to point directly to the internal server (`192.168.1.150`). When employees access `www.company.local`, the internal DNS server intercepts and resolves the name, allowing the client browser to seamlessly connect to Apache on port 80. A disruption in any single layer compromises the entire user journey: without DHCP, manual IP provisioning is required; without DNS, internal hostnames fail; and without HTTP, the destination endpoint is unreachable.

### 2. Protocol Hardening & Cryptographic Security
Unencrypted legacy protocols like standard FTP (port 21) and cleartext HTTP (port 80) expose credentials and session tokens to packet inspection on local networks. To harden these services, we enforced interface binding (`listen_address=192.168.1.150`), disabled anonymous logins, implemented chroot filesystem jails, and constrained network access using UFW packet filtering. In production enterprise architectures, transitioning to encrypted transport protocols (HTTPS via TLS 1.3 on port 443 and SFTP over SSH port 22) is required to guarantee confidentiality and data integrity.

### 3. Service Supervision & Enterprise High Availability
Supervising daemons through `systemd` ensures that core network services persist across system reboots, recover automatically from runtime crashes, and log diagnostic events in indexed binary format via `journalctl`. This reliability is essential for meeting organizational uptime goals and business continuity standards.
