## **Use Case 02: Suspicious Logon Times – After-Hours Admin Activity (T1078.004)**

**Detection of Privileged Logins Outside Business Hours**

---

### 🔧 **Lab Setup**

* **Target Machine**: Windows 10 VM (with Microsoft-linked Administrator account)
* **SIEM**: Splunk Enterprise (running on Ubuntu, Motion VM)

---

### 📚 **Objective**

Detect successful logins by privileged accounts that occur **outside standard business hours** (e.g., 9 AM – 7 PM).

---

### 🛠️ **Step-by-Step Instructions**

---

### **1. Simulate After-Hours Login (Windows 10 VM)**

1. Log in to the **Windows 10 VM** as the Administrator (Microsoft-linked account).
2. **Change system time** to simulate after-hours:

   * Open `Settings → Time & Language → Date & time`.
   * Turn off “Set time automatically”.
   * Click **Change** and set time to `03:40 AM`. Click **Change**.
3. Log out and log back in to the admin account.  This creates a successful logon event in the Security Event Log.

---

### **Step 2: Verify Logon Event Locally**

Still on Windows 10 
1. Open Event Viewer:
  - Press Win + R, type eventvwr.msc, press Enter.
2. Go to:
  - Windows Logs > Security
3. Look for Event ID 4624 (Successful Logon):


Time: Should show it occurred after hours (e.g., 7:32 PM)

![Screenshot 2025-05-28 161806](https://github.com/user-attachments/assets/f2f09657-c610-442e-aa29-f0e84c9a2fe7)

---

### **Step 3: Create Correlation Search for After-Hours Logons**
1. In Splunk Web, go to:
  - Settings > Searches, Reports, and Alerts
2. Click New Search.
3. Use the following SPL:

```spl
index=main EventCode=4624 Account_Name="suvernalathart@gmai1.com"
| where Logon_Type=2 OR Logon_Type=10 OR Logon_Type=11
| eval hour=strftime(_time, "%H")
| where hour < 9 OR hour > 19
| table _time, Account_Name, Logon_Type, Source_Network_Address, host, hour
```
![Screenshot 2025-05-28 160936](https://github.com/user-attachments/assets/3f9b65e2-b47c-4568-a325-2a027451c18e)
![Screenshot 2025-05-28 161053](https://github.com/user-attachments/assets/bdd80c8a-9d85-4232-9272-ff87a657047a)


4. Name the search: After-Hours RDP Logon
5. Save the search.












