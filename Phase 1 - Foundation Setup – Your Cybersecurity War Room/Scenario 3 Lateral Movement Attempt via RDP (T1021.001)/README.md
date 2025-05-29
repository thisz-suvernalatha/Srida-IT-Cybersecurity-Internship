## ✅ Use Case 03: Lateral Movement Attempt (RDP-style)

### 🔍 Objective:

Detect a simulated lateral movement attempt involving RDP-style login behavior — specifically, failed login attempts followed by a successful login — from a remote attacker (Kali Linux) to a target (Windows 10 VM).

---

### 🖥️ Lab Setup Recap:

* **Attacker Machine:** Kali Linux 3 
* **Target Machine:** Windows 10 VM 
* **Monitoring & Detection:** Splunk Enterprise running on Ubuntu 

---

### ✅ Step 1: Simulate Lateral Movement from Kali Linux

On **Kali Linux**, use `crackmapexec` to simulate failed and successful SMB authentications.

Run the following in the terminal:

```bash
crackmapexec smb <victor-ip> -u testuser -p 'wrongpass'   # Simulate failed auth
crackmapexec smb <victor-ip> -u testuser -p 'pass123'     # Simulate successful auth
```

🔹 Replace `<victor-ip>` with the actual IP address of your Windows 10 VM.

![Screenshot 2025-05-28 215433](https://github.com/user-attachments/assets/5c370aaf-abac-410e-8cda-0ac644f5a44f)


This simulates lateral movement — an attacker trying to authenticate to another machine using a guessed or stolen password.

---

### ✅ Step 2: Verify Logon Events on the Target 

1. On the **Windows 10 VM (Victor)**, open **Event Viewer**:

   * Press `Win + R`, type `eventvwr.msc`, and hit Enter.

2. Navigate to:

   ```
   Windows Logs → Security
   ```

3. Filter for relevant Event IDs:

   * Click on **"Filter Current Log..."**
   * In the **Event IDs** box, enter: `4625, 4624`
   * Click **OK**

4. In the event details, look for:

   * `Event ID 4625`: Failed Logon
   * `Event ID 4624`: Successful Logon
   * `Logon Type: 10` (indicates remote/interactive login like RDP or SMB)
   * `TargetUserName`: testuser
   * `Source Network Address`: IP of your Kali Linux 3 machine

🧠 This confirms the simulated attack behavior is captured by the Windows event logs.

![Screenshot 2025-05-28 220353](https://github.com/user-attachments/assets/cd8c14ff-1959-4517-955d-5746adefc9ae)
![Screenshot 2025-05-28 220415](https://github.com/user-attachments/assets/64953e80-4de0-4e0f-aa97-9104089594ec)

---

### ✅ Step 3: Detection via SIEM

Now, on **Splunk Enterprise (**, create a search to detect these behaviors.

![Screenshot 2025-05-28 221340](https://github.com/user-attachments/assets/35dd2083-bbe0-417a-a859-38967be0614a)

