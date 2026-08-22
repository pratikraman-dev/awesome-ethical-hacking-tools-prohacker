# Nmap Network Scanning Cheat Sheet

A comprehensive, copy-pasteable reference guide for Nmap (Network Mapper) commands, scanning flags, and host discovery techniques.

---

## 1. Target Specification

Nmap accepts target specifications in IP address format, CIDR notation, hostname format, or input files.

| Command | Description |
| :--- | :--- |
| `nmap 192.168.1.1` | Scan a single target IP |
| `nmap 192.168.1.1 192.168.1.2` | Scan multiple target IPs |
| `nmap 192.168.1.1-254` | Scan a range of IP addresses |
| `nmap 192.168.1.0/24` | Scan an entire subnet (CIDR notation) |
| `nmap -iL targets.txt` | Scan targets listed in a text file |
| `nmap --exclude 192.168.1.100` | Exclude specific target from the scan |

---

## 2. Host Discovery (Ping Scans)

These commands check if hosts are alive without scanning open ports.

*   **List Scan (No Scan):**
    ```bash
    nmap -sL 192.168.1.0/24
    ```
    *Simply lists target names/IPs without sending packets.*
*   **No Port Scan (Ping only):**
    ```bash
    nmap -sn 192.168.1.0/24
    ```
    *Checks if hosts are up using ICMP requests, TCP ACK, and TCP SYN.*
*   **ICMP Echo Request Ping:**
    ```bash
    nmap -PE -sn 192.168.1.0/24
    ```
*   **TCP SYN Ping (Port 80/443):**
    ```bash
    nmap -PS80,443 -sn 192.168.1.0/24
    ```
*   **ARP Ping (Local Network):**
    ```bash
    nmap -PR -sn 192.168.1.0/24
    ```

---

## 3. Port Scanning Techniques

| Flag | Scan Type | Description | Requirement |
| :--- | :--- | :--- | :--- |
| `-sS` | **TCP SYN Scan** | Stealthy half-open scan; does not complete TCP handshake. | Root/Admin |
| `-sT` | **TCP Connect Scan** | Completes TCP handshake; slower, logs connection. | User privileges |
| `-sU` | **UDP Scan** | Scans UDP ports (DNS, DHCP, SNMP); slow. | Root/Admin |
| `-sA` | **TCP ACK Scan** | Maps firewall rulesets; detects filtered ports. | Root/Admin |
| `-sF` | **FIN Scan** | Sends FIN packet; bypasses stateless firewalls. | Root/Admin |

---

## 4. Port Specifications

Customize which ports Nmap scans. By default, Nmap scans the top 1,000 most common ports.

*   **Scan specific port:**
    ```bash
    nmap -p 80 target.com
    ```
*   **Scan range of ports:**
    ```bash
    nmap -p 1-1024 target.com
    ```
*   **Scan all 65,535 ports:**
    ```bash
    nmap -p- target.com
    ```
*   **Scan top 100 ports (Fast scan):**
    ```bash
    nmap -F target.com
    ```
*   **Scan specific ports by name:**
    ```bash
    nmap -p http,https target.com
    ```

---

## 5. Service & OS Detection

Identify software versions running on open ports and fingerprint the target Operating System.

*   **Service Version Detection:**
    ```bash
    nmap -sV target.com
    ```
*   **OS Fingerprinting:**
    ```bash
    nmap -O target.com
    ```
*   **Aggressive Scan:**
    ```bash
    nmap -A target.com
    ```
    *Enables OS detection, service version detection, script scanning, and traceroute.*

---

## 6. Scan Performance & Timing (`-T0` to `-T5`)

Nmap timing templates adjust speed to bypass IDS/firewalls or speed up scanning.

```
<- Slower (Stealthy)                                    Faster (Noisy) ->
[ -T0 (Paranoid) ] [ -T1 (Sneaky) ] [ -T2 (Polite) ] [ -T3 (Normal) ] [ -T4 (Aggressive) ] [ -T5 (Insane) ]
```

*   **Recommended for CTFs & Labs:**
    ```bash
    nmap -T4 target.com
    ```
*   **Stealth scan to avoid detection:**
    ```bash
    nmap -T1 target.com
    ```

---

## 7. Nmap Scripting Engine (NSE)

Nmap contains over 600 built-in scripts to detect vulnerabilities, brute-force logins, and gather metadata.

*   **Scan with default safe scripts:**
    ```bash
    nmap -sC target.com
    ```
*   **Scan for vulnerabilities:**
    ```bash
    nmap --script vuln target.com
    ```
*   **Brute force SSH credentials:**
    ```bash
    nmap -p 22 --script ssh-brute target.com
    ```
*   **Find HTTP directories (web fuzzing):**
    ```bash
    nmap -p 80 --script http-enum target.com
    ```

---

## 8. Nmap Output Formats

Save Nmap results to files for documentation, reporting, or parsing.

*   **Normal text output:**
    ```bash
    nmap -oN scan_results.txt target.com
    ```
*   **XML format (for importing to Burp/Metasploit):**
    ```bash
    nmap -oX scan_results.xml target.com
    ```
*   **Grepable format (easy parsing with grep/awk):**
    ```bash
    nmap -oG scan_results.gnmap target.com
    ```
*   **All three major formats (highly recommended):**
    ```bash
    nmap -oA target_scan target.com
    ```
    *Saves files as target_scan.nmap, target_scan.xml, and target_scan.gnmap.*

---

## 9. Disclaimer

> [!WARNING]
> Scanning ports without authorization is considered a precursor to an attack. Ensure you have written permission before scanning any target network.
