# Detecting Command & Control (C2) Beaconing 
## Overview

This guide walks you step-by-step through simulating and detecting Command & Control (C2) beaconing behavior in a lab setup.
---

## 1. Lab Setup

### Virtual Machines Used:

1. **Ubuntu VM**:

   * Runs Splunk Enterprise
2. **Windows 10 VM**:

   * Acts as the victim
   * Has Splunk Universal Forwarder, Sysmon, and PowerShell beaconing script
3. **Kali Linux VM**:

   * Acts as the attacker (C2 server)

---

## 2. Goal of the Simulation

Simulate a basic C2 scenario where the victim machine (Windows 10) makes repeated HTTP requests (every 60 seconds) to the attacker's IP (Kali Linux), and detect this activity in Splunk.

---

## 3. Attacker Setup (Kali Linux)

### Step 1: Start a simple HTTP server

```bash
sudo python3 -m http.server 80
```

This will run an HTTP server on port 80 to simulate a C2 endpoint.

Ensure the server starts successfully:

```
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/)
```

---

## 4. Victim Setup (Windows 10)

### Step 1: Install and Configure Sysmon

Use SwiftOnSecurity Sysmon config or your own to ensure Event ID 3 (network connections) are logged.

### Step 2: Beaconing Script (PowerShell)

Create and run this PowerShell script:

```powershell
$code = {
    while ($true) {
        Invoke-WebRequest http://<KALI_IP>/ping
        Start-Sleep -Seconds 60
    }
}
Start-Job -ScriptBlock $code
```

Replace `<KALI_IP>` with your actual Kali IP (e.g., `192.168.246.131`).

Check with `Get-Job` to confirm it's running.

---

## 5. Splunk Setup (Ubuntu)

### Step 1: Verify Logs in Splunk

Open Splunk Web: `http://<ubuntu-ip>:8000`


Look for destination IP `192.168.246.131` and the process `powershell.exe`.

