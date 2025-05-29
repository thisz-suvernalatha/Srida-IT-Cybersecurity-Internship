# 🛡️ Use Case 05: Hidden User Account Creation (T1136)

### 🎯 Goal:
Detect when:
  A new local user account is created (Event ID 4720).
  That account is added to a privileged group (Event ID 4728 or 4732). 
  Especially if this happens outside of business hours or the account is suspiciously named.

## 🧪 Lab Setup

| Component        | Role                | Key Tools                               |
| ---------------- | ------------------- | --------------------------------------- |
| 🪟 Windows 10 VM | Victim machine      | Splunk Universal Forwarder, Sysmon      |
| 🐧 Ubuntu VM     | Splunk Server       | Splunk Enterprise                       |

---

## ✅ Step 1: Simulate the Attack

### On your Windows 10 VM:

Open **Command Prompt as Administrator**, then run:

```cmd
net user support1 Password123! /add
net localgroup administrators support1 /add
```

🔹 This creates a suspicious user account (`support1`)
🔹 Then adds it directly to the **Administrators** group

![Screenshot 2025-05-29 162814](https://github.com/user-attachments/assets/49eb91f9-9758-409e-b6f6-4dcd7949aa46)
![Screenshot 2025-05-29 164452](https://github.com/user-attachments/assets/ddc77043-520a-47d4-a18a-0a1bcbda27a8)

---

## ✅ Step 2: Check Event Logs Locally

### Open Event Viewer on Windows:

1. Press `Win + R`, type: `eventvwr`, press Enter
2. Go to `Windows Logs > Security`
3. Click **"Filter Current Log"**
4. Enter Event IDs: `4720, 4732` and click OK

🔍 Look for:

* Event 4720: Local user created
* Event 4732: User added to Admin group

![Screenshot 2025-05-29 165318](https://github.com/user-attachments/assets/f74cf5dc-3fd9-484b-a5f0-5835dfe013e3)


---

## ✅ Step 3: Confirm Log Ingestion in Splunk

### On Splunk Web (Ubuntu VM):

1. Go to **Search & Reporting**
2. Run the following searches:

#### 🔎 User Account Created (Event ID 4720)

```spl
index=* EventCode=4720
| table _time, Account_Name, SubjectUserName, Message
| sort -_time
```

#### 🔎 Privilege Escalation (Event ID 4732)

```spl
index=* EventCode=4732
| table _time, GroupName, SubjectUserName, MemberName, Message
| sort -_time
```

📌 You should see:

* The `support1` account created
* It being added to the Administrators group

---

## Summary
* Simulated the attack
* Verified the events in **Event Viewer**
* Verified ingestion and visibility in **Splunk**

