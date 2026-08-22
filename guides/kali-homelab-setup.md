# Guide: Setting Up a Local Hacking Lab

Learn how to configure a safe, isolated, and virtualized hacking environment using VirtualBox, Kali Linux, and vulnerable target machines.

---

## Lab Network Topology

```mermaid
graph TD
    subgraph Host Machine (Physical Computer)
        HostOS["Physical Host Operating System"]
        Hypervisor["Hypervisor (VirtualBox / VMware Switch)"]
        
        subgraph Isolated Virtual NAT Network (10.0.2.0/24)
            Kali["Kali Linux (Attacker VM) <br> IP: 10.0.2.4"]
            Metasploit["Metasploitable 2 (Target VM) <br> IP: 10.0.2.5"]
            JuiceShop["Docker: OWASP Juice Shop <br> Port: 3000"]
        end
    end

    Kali -->|Network Scan & Exploit| Metasploit
    Kali -->|HTTP Web Hacking| JuiceShop
    Hypervisor --- HostOS
    Hypervisor === Isolated Virtual NAT Network
```

---


## Why Set Up a Virtual Lab?

> [!IMPORTANT]
> Running security scanning and exploitation tools against live internet targets is illegal. A local lab lets you practice offensive security techniques safely, offline, and legally without disrupting public systems.

---

## Step 1: Install a Hypervisor

A hypervisor allows you to run multiple guest operating systems (VMs) on a single physical host machine.

1.  Download and install **[Oracle VM VirtualBox](https://www.virtualbox.org/)** (Free, Open Source) or **[VMware Workstation Player](https://www.vmware.com/products/workstation-player.html)** (Free for personal use).
2.  Install the corresponding **VirtualBox Extension Pack** (for USB 2.0/3.0 support and screen resolution scaling).

---

## Step 2: Set Up Kali Linux (Attacker Machine)

Kali Linux is the standard distribution preloaded with penetration testing tools.

1.  Go to the [Kali Linux Virtual Machines page](https://www.kali.org/get-kali/#kali-virtual-machines).
2.  Download the pre-packaged VirtualBox or VMware image (`.ova` or `.7z`).
3.  **Import the VM:**
    *   *In VirtualBox:* Go to **File -> Import Appliance**, select the `.ova` file, and click **Import**.
4.  Start the VM. The default credentials are:
    *   **Username:** `kali`
    *   **Password:** `kali`

---

## Step 3: Set Up Target Machines (Vulnerable VMs)

To practice hacking, you need vulnerable systems to attack.

### A. Metasploitable 2 (Linux Target)
Metasploitable is an intentionally vulnerable Ubuntu-based virtual machine designed for testing network tools and exploiting common software services.
1.  Download Metasploitable 2 from [SourceForge](https://sourceforge.net/projects/metasploitable/).
2.  Decompress the `.zip` file to reveal the `.vmdk` virtual disk.
3.  In VirtualBox, click **New** to create a VM:
    *   **Type:** Linux, **Version:** Ubuntu (64-bit).
    *   **RAM:** 512 MB or 1 GB.
    *   **Hard disk:** Select **Use an existing virtual hard disk file** and browse to the extracted `.vmdk` file.
4.  Start the VM. Default login is:
    *   **Username:** `msfadmin`
    *   **Password:** `msfadmin`

### B. OWASP Juice Shop (Web Vulnerability Target)
Juice Shop is a modern Node.js application containing security flaws spanning the entire OWASP Top 10.
You can run it inside Docker on your Kali Linux VM:
```bash
# 1. Install Docker on Kali
sudo apt update && sudo apt install docker.io -y

# 2. Start Docker service
sudo systemctl start docker

# 3. Pull and run OWASP Juice Shop
sudo docker run -d -p 3000:3000 bkimminich/juice-shop
```
*Access the target in your browser at `http://localhost:3000`.*

---

## Step 4: Configure Network Isolation (Crucial)

To prevent your vulnerable target VMs from being exposed to your home Wi-Fi network (or the public internet), you must put them on an isolated virtual network.

### In VirtualBox (NAT Network)
1.  In VirtualBox, go to **File -> Tools -> Network Manager** (or Preferences -> Network).
2.  Under **NAT Networks**, click **Create**.
3.  Verify the settings:
    *   **Network Name:** `HackingLab`
    *   **Network CIDR:** `10.0.2.0/24`
    *   Ensure **Supports DHCP** is enabled.
4.  Apply changes.
5.  Go to the settings of **both** your Kali Linux VM and Metasploitable VM:
    *   Navigate to **Network -> Adapter 1**.
    *   Set **Attached to:** **NAT Network**.
    *   Set **Name:** **HackingLab**.
6.  Restart both virtual machines. They will now be on the same private sub-network and can ping each other, but targets are blocked from the internet.

---

## Step 5: Verify Connectivity

1.  Open terminal on your **Kali Linux** VM.
2.  Find Kali's IP address:
    ```bash
    ip a
    ```
    *Look for the `eth0` interface IP (e.g., `10.0.2.4`).*
3.  Perform a quick ping scan using Nmap to find the IP of Metasploitable:
    ```bash
    nmap -sn 10.0.2.0/24
    ```
4.  Verify you can ping the target:
    ```bash
    ping -c 3 <MetasploitableIP>
    ```

You now have a fully operational and isolated local penetration testing lab environment!
