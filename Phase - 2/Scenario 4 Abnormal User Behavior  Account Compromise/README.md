# 🛡️ **Abnormal User Behavior / Account Compromise**

### 🎯 Objective:

Detect when a valid user logs in at **odd hours** and accesses **100+ files quickly** — a common indicator of **insider threats** or **compromised credentials**.

---

## 🧪 Lab Setup

| VM Role                  | OS                       | Purpose                                                   |
| ------------------------ | ------------------------ | --------------------------------------------------------- |
| 💻 Windows 10            | Victim                   | Host logs suspicious activity                             |
| 🐧 Ubuntu                | Splunk Enterprise Server | Collect and analyze logs                                  |
| ⚔️ Kali Linux (optional) | Attacker                 | Simulate external threat (not required for this use case) |

---

1. ✅ **Splunk Universal Forwarder** installed on Windows 10.
2. ✅ **Sysmon** configured to log Event ID 11 (FileCreate).
3. ✅ Windows Event Logs (Security) forwarded to Splunk.
4. ✅ Windows audit policy set to log:

   * **Logon events** (Event ID 4624)
   * **File share accesses** (Event ID 5145) *(if Sysmon not used)*

---

## 🔧 Step-by-Step Attack Simulation

### ✅ Step 1: Simulate Logon at Odd Hours

On the **Windows 10** machine:

1. Open Command Prompt as admin.
2. Change the system time to **2:15 AM**:

   ```cmd
   time 02:00:00
   ```
3. Log off and back in as a valid user.
4. This generates an **Event ID 4624** at an unusual time.

---

### ✅ Step 2: Simulate Mass File Access

Run this in PowerShell on the same Windows 10 machine:

```powershell
robocopy C:\Windows\System32\drivers C:\Users\Public\TempCopy /E /NFL /NDL /NJH /NJS /nc /ns /np
```

> Simulates accessing 100+ files in seconds.


