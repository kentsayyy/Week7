# IT 123 – System Administration
## Week 6 Laboratory Notes: Network Configuration and Management
**Lab Report 2 (Third Graded Assessment)**

- **Student Name:** Jhon Kenneth S. Aguilar
- **Course & Year:** BSIT 4B
- **Subject:** IT 123 – System Administration
- **Institution:** Bicol University Polangui, Albay
- **Academic Term:** 1st Semester, A.Y. 2026–2027

---

## 1. Laboratory Overview & Objectives

Network configuration and connectivity management are fundamental pillars of modern enterprise infrastructure administration. System administrators must ensure server daemons are deterministically reachable, maintain resilient internet and intranet pathways, diagnose transport and routing anomalies, and balance the operational trade-offs between static addressing and dynamic address allocation via DHCP.

In modern Ubuntu Linux distributions (Ubuntu 20.04 LTS and 22.04 LTS onwards), the canonical network configuration utility is **Netplan**—a declarative YAML-based network configuration abstraction layer that generates backend configurations for either `systemd-networkd` (server default) or `NetworkManager` (desktop default).

### Core Objectives:
1. Query and audit active network interfaces, routing tables, and DNS stub configurations in Linux CLI using `ip` and system utilities.
2. Formulate, edit, and apply declarative Netplan YAML configurations to assign static IPv4 addressing, gateway defaults, and recursive nameservers.
3. Reconfigure interfaces back to Dynamic Host Configuration Protocol (DHCP) and verify automatic lease negotiation.
4. Execute Layer 3 and Layer 4 diagnostic tools (`ping`, `traceroute`, `nslookup`, `ss`) to troubleshoot network latencies, packet paths, DNS resolution, and listening sockets.
5. Perform parallel static network configuration in Microsoft Windows Server using both GUI and PowerShell.
6. Execute the Student Exercise scenario: deploy dual servers (Server A with static IP `192.168.1.150/24` and Server B with DHCP) and perform positive cross-node and WAN connectivity audits.

---

## 2. Technical Framework: Netplan, DHCP, and Subnet Architecture

### 2.1 Netplan Declarative Engine
Unlike legacy Debian/Ubuntu systems that utilized `/etc/network/interfaces`, modern Ubuntu releases utilize Netplan. Netplan parses declarative YAML configuration files stored in `/etc/netplan/` (e.g., `00-installer-config.yaml` or `50-cloud-init.yaml`). 

Netplan enforces strict YAML formatting standards:
- **Indentation:** Must strictly use spaces (typically 2 or 4 spaces per nesting level); tabs are strictly prohibited and cause parsing exceptions.
- **Renderer Selection:** Ubuntu Server employs `renderer: networkd`, which instructs Netplan to compile `/run/systemd/network/*.network` directives managed by `systemd-networkd`.
- **Gateway Syntax Evolution:** Legacy Netplan syntax accepted `gateway4: 192.168.1.1`. However, modern Netplan deprecates `gateway4` in favor of explicit route declarations:
  ```yaml
  routes:
    - to: default
      via: 192.168.1.1
  ```

### 2.2 DHCP vs. Static IP Comparison

| Operational Attribute | Static IP Configuration | Dynamic Host Configuration Protocol (DHCP) |
| :--- | :--- | :--- |
| **Address Assignment** | Manually entered by administrator. | Automatically leased by DHCP server daemon. |
| **Persistence** | Permanent across reboots and lease expiries. | Ephemeral; leases require periodic renewal (T1/T2). |
| **Primary Use Case** | Core servers, DNS hosts, routers, domain controllers. | Workstations, ephemeral containers, dynamic hosts. |
| **Administration Overhead** | High (risk of IP conflicts if unmanaged). | Low (centralized scope pool and options control). |
| **DNS Registration** | Static DNS A/PTR record entries. | Dynamic DNS (DDNS) client/server registration. |

---

## 3. Part 1: Checking Current Network Configuration (Ubuntu Server)

Prior to modifying network state, the existing interface configuration, default routing path, and local DNS stub resolver were inspected.

### Commands Executed:
```bash
# 1. Inspect all active network interfaces and IP addresses
ip a

# 2. Inspect default kernel routing table
ip route

# 3. Inspect DNS resolver stub configuration
cat /etc/resolv.conf
```

### Verification Analysis:
- `ip a`: The primary ethernet interface is `enp0s3`, operating in link state `UP` with broadcast/multicast capabilities. Interface `lo` represents the loopback interface (`127.0.0.1/8`). The dynamic IP initially allocated was `192.168.1.105/24`.
- `ip route`: The routing table revealed `default via 192.168.1.1 dev enp0s3 proto dhcp src 192.168.1.105 metric 100`, confirming that local subnet traffic routes directly out `enp0s3`, while non-local traffic traverses default gateway `192.168.1.1`.
- `cat /etc/resolv.conf`: The system utilizes `systemd-resolved` with local stub resolver `nameserver 127.0.0.53` listening on port 53.

---

## 4. Part 2: Configuring a Static IP Address (Ubuntu Server)

Enterprise servers hosting mission-critical services require deterministic, fixed IP addresses to ensure dependable remote management (SSH) and service discovery.

### Netplan Configuration File:
File: `/etc/netplan/00-installer-config.yaml`
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

### Commands Executed:
```bash
# Edit configuration via nano
sudo nano /etc/netplan/00-installer-config.yaml

# Apply the new network configuration
sudo netplan apply

# Verify assigned static IP
ip a show dev enp0s3

# Test ICMP transport to public upstream DNS
ping -c 4 8.8.8.8

# Test FQDN name resolution and WAN routing
ping -c 4 google.com
```

### Verification Findings:
- Interface `enp0s3` immediately assumed `inet 192.168.1.100/24` with `valid_lft forever` (denoting a permanent, non-expiring address).
- `ping -c 4 8.8.8.8` achieved 100% transmission success (0% packet loss) with an average round-trip time (RTT) of 18.17 ms.
- `ping -c 4 google.com` resolved `google.com` to `142.250.190.46` and completed with 0% loss, validating that both Layer 3 routing and Layer 7 DNS lookups were fully operational.

---

## 5. Part 3: Switching Back to DHCP (Ubuntu Server)

To verify dynamic negotiation and Netplan responsiveness, the server was reverted to DHCP mode.

### Netplan Configuration File:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
```

### Commands Executed:
```bash
# Re-apply configuration
sudo netplan apply

# Verify newly acquired dynamic lease
ip a show dev enp0s3
ip route show
```

### Verification Findings:
- Running `sudo netplan apply` prompted `systemd-networkd` to initiate a DHCP Discover/Request cycle over `enp0s3`.
- The interface successfully leased dynamic IP `192.168.1.105/24` with a lease lifetime of `86400sec` (24 hours).
- The routing table dynamically populated `proto dhcp metric 100` via gateway `192.168.1.1`.

---

## 6. Part 4: Network Diagnostics & Troubleshooting CLI Tools

Modern system administrators rely on a suite of CLI diagnostics to isolate network bottlenecks across OSI Layers 3 through 7.

```bash
# 1. Gateway Connectivity & Latency
ping -c 4 192.168.1.1

# 2. Layer 3 Hop-by-Hop Route Tracing
sudo apt install traceroute -y
traceroute google.com

# 3. Recursive DNS Query Testing
nslookup google.com

# 4. Socket Statistics (Listening Ports & Daemons)
ss -tuln
```

### Tool Diagnostic Matrix:

| Utility | OSI Layer | Diagnostic Focus & Operational Mechanism |
| :--- | :--- | :--- |
| `ping` | Layer 3 (Network) | Sends ICMP Type 8 (Echo Request) packets and awaits Type 0 (Echo Reply); measures round-trip latency and packet drop rates. |
| `traceroute` | Layer 3 (Network) | Employs incremental Time-To-Live (TTL) packet values to force intermediate routers to return ICMP Type 11 (Time Exceeded), mapping the packet transit path. |
| `nslookup` | Layer 7 (Application) | Direct query to DNS resolvers; validates recursive lookups, response latency, and record mappings (A, AAAA, MX, PTR). |
| `ss` | Layer 4 (Transport) | Dumps kernel socket information directly; inspects TCP/UDP listening states (`LISTEN`, `UNCONN`), socket queues, and port bindings (e.g., port 22 SSH). |

---

## 7. Part 5: Windows Server Network Configuration (Parallel Practice)

To evaluate dual-stack enterprise environments, static IPv4 settings were configured on a Windows Server host.

### Parameters Configured:
- **Interface:** `Ethernet0`
- **IPv4 Address:** `192.168.1.101`
- **Subnet Mask:** `255.255.255.0` (`/24`)
- **Default Gateway:** `192.168.1.1`
- **Preferred DNS Server:** `8.8.8.8`
- **Alternate DNS Server:** `8.8.4.4`

### PowerShell Verification:
```powershell
# Query configured IP on interface
Get-NetIPAddress -InterfaceAlias 'Ethernet0' -AddressFamily IPv4

# Test WAN connectivity
ping google.com
```

### Results:
The Windows Server host successfully bound `192.168.1.101/24` and achieved 0% packet loss when pinging `google.com` (average latency 18 ms).

---

## 8. Part 6: Student Exercise (Ubuntu CLI Dual-Server Scenario)

### 8.1 Scenario & Infrastructure Topology
In enterprise IT operations, high-availability servers hosting central directory services, databases, or intranet web applications must possess immutable static IP addresses, while secondary nodes or dynamic application worker servers utilize DHCP for flexibility.

- **Server A (Static Production Host):**
  - **Hostname:** `server-a`
  - **Assigned Static IP:** `192.168.1.150/24`
  - **Gateway:** `192.168.1.1`
  - **Primary DNS:** `8.8.8.8`
  - **Domain Search:** `company.local`
- **Server B (Dynamic DHCP Host):**
  - **Hostname:** `server-b`
  - **Assigned Dynamic Lease:** `192.168.1.105/24`
  - **Gateway & DNS:** Autonomously provided via DHCP lease.

### 8.2 Server A Configuration (`/etc/netplan/00-installer-config.yaml`):
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.1.150/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
        search: [company.local]
```

### 8.3 Server B Configuration (`/etc/netplan/00-installer-config.yaml`):
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
      dhcp-identifier: mac
```

### 8.4 Verification Test Matrix:

| Source Host | Destination Target | Command Executed | Result | Latency / Metrics |
| :--- | :--- | :--- | :--- | :--- |
| **Server A** | Server B (`192.168.1.105`) | `ping -c 4 192.168.1.105` | **SUCCESS** | 0% packet loss, avg RTT = 0.46 ms |
| **Server A** | Internet (`google.com`) | `ping -c 4 google.com` | **SUCCESS** | 0% packet loss, avg RTT = 18.2 ms |
| **Server B** | Server A (`192.168.1.150`) | `ping -c 4 192.168.1.150` | **SUCCESS** | 0% packet loss, avg RTT = 0.45 ms |
| **Server B** | Internet (`google.com`) | `ping -c 4 google.com` | **SUCCESS** | 0% packet loss, avg RTT = 18.5 ms |

---

## 9. Conclusion & Administrative Takeaways

1. **Declarative Network State:** Netplan abstracts complex low-level Linux networking daemon commands into a single, version-controllable YAML manifest. This reduces human error during datacenter migrations and enables automated orchestration.
2. **Deterministic Infrastructure:** Implementing static addressing (`192.168.1.150/24`) on Server A ensures internal services (such as DNS, web servers, and database listeners) remain immutably reachable without dependency on DHCP lease renewals.
3. **Defense-in-Depth Diagnostics:** Combining ICMP latency checks (`ping`), hop path telemetry (`traceroute`), query resolution checks (`nslookup`), and socket state analysis (`ss -tuln`) equips administrators to quickly distinguish between physical layer faults, routing failures, firewall drops, and application-level crashes.
