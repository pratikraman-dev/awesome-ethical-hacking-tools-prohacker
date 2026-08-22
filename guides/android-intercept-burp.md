# Guide: Intercepting Android App Traffic with Burp Suite

A step-by-step tutorial on setting up a proxy, installing a custom CA certificate on Android, and using Frida to bypass SSL Pinning so you can inspect mobile API requests.

---

## Prerequisites

Before starting, ensure you have the following installed and configured:
1.  **Burp Suite Community/Professional** installed on your host machine.
2.  **Android Emulator** (e.g., Genymotion, Android Studio Emulator) or a **Rooted Android Device** connected via ADB.
3.  **ADB (Android Debug Bridge)** command-line utility installed.
4.  **Python 3** and **Frida-tools** installed on your host machine (`pip install frida-tools`).

---

## Step 1: Configure Burp Suite Proxy

We need to configure Burp Suite to listen on all interfaces so the Android device can connect.

1.  Open Burp Suite and navigate to **Proxy -> Proxy settings**.
2.  Under **Proxy listeners**, click **Add**.
3.  Set the **Bind to port** to `8082` (or any free port).
4.  Set **Bind to address** to **All interfaces** (or select your local Wi-Fi IP address).
5.  Click **OK** and ensure the listener checkbox is checked.

---

## Step 2: Configure Android Device Network Proxy

1.  Find the local IP address of your host machine running Burp Suite (e.g., `192.168.1.15`).
2.  On the Android device, go to **Settings -> Network & Internet -> Wi-Fi**.
3.  Long-press the connected network name and select **Modify Network** (or click the edit pencil icon).
4.  Change the **Proxy** option from *None* to *Manual*.
5.  Set **Proxy hostname** to your host machine's IP (e.g., `192.168.1.15`).
6.  Set **Proxy port** to `8082`.
7.  Click **Save**.

---

## Step 3: Install Burp CA Certificate on Android

Since Android 7+ (API 24), applications by default do not trust user-installed CA certificates. We need to install the certificate as a **System Certificate** (requires root).

### A. Export the Certificate from Burp
1.  On your host machine, open a browser and go to `http://burpsuite` (while proxy is enabled).
2.  Click **CA Certificate** in the top right corner to download `cacert.der`.

### B. Convert DER to PEM Format
Android requires the certificate to be named with its subject hash in `.0` format. Use `openssl` to convert it:

```bash
# 1. Convert DER to PEM
openssl x509 -inform DER -in cacert.der -out cacert.pem

# 2. Get the certificate's subject hash
openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -n 1
```
*Note the 8-character output hash (e.g., `9a5ba575`).*

Rename the PEM certificate to match this hash:
```bash
mv cacert.pem 9a5ba575.0
```

### C. Upload Certificate to Android System Store
Using ADB, push the certificate to the system CA store directory:

```bash
# 1. Restart ADB as root
adb root

# 2. Remount the /system partition as writable
adb remount

# 3. Push the certificate to the device
adb push 9a5ba575.0 /system/etc/security/cacerts/

# 4. Correct permissions of the certificate file
adb shell "chmod 644 /system/etc/security/cacerts/9a5ba575.0"

# 5. Reboot the device to apply changes
adb reboot
```

---

## Step 4: Bypass SSL Pinning with Frida

Many modern Android applications implement **SSL Pinning** (hardcoding the trusted certificate inside the app binary), which blocks Burp Suite even with a system certificate. We use Frida to hook and disable this validation at runtime.

### A. Start Frida Server on Android Device
1.  Check your Android CPU architecture:
    ```bash
    adb shell getprop ro.product.cpu.abi
    ```
2.  Download the matching `frida-server` binary from the [Frida Github Releases](https://github.com/frida/frida/releases).
3.  Decompress and upload it to the device:
    ```bash
    adb push frida-server /data/local/tmp/
    adb shell "chmod 755 /data/local/tmp/frida-server"
    ```
4.  Run frida-server in the background as root:
    ```bash
    adb shell "su -c /data/local/tmp/frida-server &"
    ```

### B. Inject the SSL Pinning Bypass Script
1.  Find the package name of the target application (e.g., `com.target.app`):
    ```bash
    frida-ps -Uai
    ```
2.  Run Frida, injecting a universal bypass script (such as those from [Codeshare](https://codeshare.frida.re/)):
    ```bash
    frida -U --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -f com.target.app
    ```
3.  Frida will launch the app, hook the SSL/TLS sockets, and bypass the pinning verification.

You should now see HTTP/S traffic flowing through **Burp Suite's HTTP History** tab!

---

## Disclaimer

> [!WARNING]
> Only intercept traffic on applications you own or have explicit authorization to audit. Intercepting third-party applications without permission may violate Terms of Service and data protection regulations.
