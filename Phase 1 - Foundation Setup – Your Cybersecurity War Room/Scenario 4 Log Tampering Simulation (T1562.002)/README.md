## ✅ **Use Case 04: Log Tampering Detection (T1562.002)**


### 🧪 **Lab Setup Overview**

* **Attacker**: Kali Linux
* **Victim**: Windows 10 VM
* **Monitoring**: Splunk Enterprise on Ubuntu (receives logs via Universal Forwarder from victim)

---

### 🪛 Step-by-Step Walkthrough

---

### ✅ Step 1: Confirm Logging is Working

#### 🧩 What and Why:

You need to make sure Windows is actually generating and forwarding the logs we care about.

#### 🔍 Checkpoints:

* Sysmon is installed and sending logs (`Event ID 1` = process creation)
* PowerShell script block logging is enabled (for `Event ID 4104`)
* Security log forwarding is enabled (for `Event ID 1102`)

---

### ✅ Step 2: Simulate the Attack


Run commands on the Windows VM that simulate an attacker trying to cover their tracks.

#### 💣 Commands Used:

From **Remote Windows Session**:
![Screenshot 2025-05-29 134259](https://github.com/user-attachments/assets/b9c30217-7585-40bf-984b-057225d084b5)
![Screenshot 2025-05-29 134241](https://github.com/user-attachments/assets/4e2c9992-d39b-4458-91d3-7d9af8db57f9)

```cmd
wevtutil cl Security
```
![Screenshot 2025-05-29 134448](https://github.com/user-attachments/assets/7d271eac-544a-470d-a127-76c2073ea460)

From **PowerShell (as admin)**:

```powershell
Clear-EventLog -LogName Security
```
![Screenshot 2025-05-29 140048](https://github.com/user-attachments/assets/afabfeb3-699e-4910-865c-1399f8b9e960)

---

### ✅ Step 3: Confirm Detection in Splunk

#### 📌 Search 1: Detect Process Creation (Sysmon Event ID 1)

```spl
index=winevent sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 "wevtutil.exe"
```
![Screenshot 2025-05-29 135135](https://github.com/user-attachments/assets/e32f08f4-113b-4b8b-828a-db485b951783)

✅ Detected!

---

#### 📌 Search 2: Detect Security Log Clearing (Security Event ID 1102)

```spl
index=main sourcetype="WinEventLog:Security" EventCode=1102
```
![Screenshot 2025-05-29 135623](https://github.com/user-attachments/assets/974d5a08-b636-4dcc-a0e7-5fe776bbcf45)

✅ Detected!

---

#### 📌 Search 3: Detect PowerShell Script (PowerShell Event ID 4104)

```spl
index=winevent sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 "Clear-EventLog"
```
![Screenshot 2025-05-29 140107](https://github.com/user-attachments/assets/a17925cd-753b-47e0-946a-d53410bea3f5)

✅ Detected!

---

#### 📌 Bonus: Combine All 3 in One Search

```spl
(
  index=winevent sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 "wevtutil.exe"
)
OR (
  index=main sourcetype="WinEventLog:Security" EventCode=1102
)
OR (
  index=winevent sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 "Clear-EventLog"
)
| sort - _time
```
![Screenshot 2025-05-29 141928](https://github.com/user-attachments/assets/46ba903a-9c21-4b76-88ba-9458518dc41b)


✅ All three event types shown together!

---

### ✅ Step 4: View in Event Viewer (Optional, for Windows Learners)

You also checked:

* **Sysmon** logs under: `Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`
* **PowerShell** logs under: `... → PowerShell → Operational`
* **Security** logs under: `Windows Logs → Security`

![Screenshot 2025-05-29 140526](https://github.com/user-attachments/assets/215bf2c3-73b3-468b-b916-d1baefb8ad34)
![Screenshot 2025-05-29 140907](https://github.com/user-attachments/assets/08655ea5-55a5-4722-85ef-43125728b42f)
![Screenshot 2025-05-29 141041](https://github.com/user-attachments/assets/1cb222e7-536d-4479-ac97-08933b3b9677)


---

### ✅ Summary

* Simulated log tampering
* Detected it with Sysmon, PowerShell logging, and Windows Security logs
* Built Splunk queries to monitor and validate all activity
* Viewed the same logs inside Event Viewer for local confirmation

---
