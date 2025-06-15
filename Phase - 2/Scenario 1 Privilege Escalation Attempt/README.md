**Title:** Simulating Privilege Escalation Attack in a Windows Lab (MITRE T1055 / T1547)

---

**Objective:**
Simulate a basic privilege escalation attempt on a Windows 10 VM and detect it using Wazuh. This scenario mimics a local attacker creating a new user and adding them to the Administrators group.

---

* A lab with:

  * **Windows 10 VM** (victim) with Sysmon + Wazuh Agent
  * **Ubuntu VM** (Wazuh Manager + Wazuh Dashboard)
  * **Kali Linux VM** (optional attacker)
* Ensure Wazuh is receiving logs from the Windows machine
* Admin access on Windows 10

---

**Step 1: Simulate the Attack**

On the Windows 10 VM, open **Command Prompt as Administrator** and run the following:

```cmd
whoami
net user testuser /add
net localgroup administrators testuser /add
runas /user:testuser cmd
whoami
```

This will:

* Create a new user `testuser`
* Add `testuser` to the Administrators group
* Open a new shell as `testuser`

---

**Step 2: What Events to Look For**

The following **Windows Event IDs** are typically generated:

* `4720` – User account created
* `4728` – User added to a security-enabled global group (Administrators)
* `4672` – Special privileges assigned to new logon

---

**Step 3: Detect in Wazuh Dashboard**

Go to your **Wazuh Dashboard**:

1. Navigate to **Security Events**
2. In the search bar, use:

```kql
data.win.event_data.TargetUserName: "testuser"
```

Or filter by Event ID:

```kql
data.win.system.eventID: "4720" or data.win.system.eventID: "4728" or data.win.system.eventID: "4672"
```

---

**Step 4: Understanding Wazuh Rule Matches**

These logs may match default Wazuh rules like:

* `60109` – User account created/enabled (mapped to MITRE T1098)
* `60110` – User account changed

Look for:

* `rule.description`: User account created/enabled
* `rule.mitre.id`: `T1098`
* `rule.level`: Typically `8`, consider writing a custom rule to raise severity

---

