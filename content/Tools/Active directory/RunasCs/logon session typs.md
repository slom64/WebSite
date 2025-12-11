These values correspond to **Windows logon types**, which define _how_ a user is authenticated and _what security context/permissions they get_.  
They matter a lot for lateral movement, constrained delegation, and credential exposure.

Below is an **Attack-Focused Cheat Breakdown**:

Things that works with rubeus.exe
- `9`:Keep in mind. You can change the TGT ticket and inject newer ones for another users in this logon session, but in the start you can't `.\Rubeus.exe x x powershell -l 9`. you can't specify username and password of other user, the logon session will be in the same context of the user you have called Rubeus with. 
things didn't work: 8,3

---

### 🔥 `2 – Interactive`

User logs on _locally_ to workstation/console.

Use when:  
✔ GUI access / RDP console login  
✔ Mimikatz dumps local creds (LSA stores full tokens)

⚠ High credential exposure — credentials are stored in LSASS.

---

### 🔥 `3 – Network`

Used for remote authentication where plaintext password is **not** sent.  
Example: SMB, LDAP, WinRM (Kerberos/NTLM challenge-response).

Use when:  
✔ You want remote access without password stored in memory  
✔ Psexec, SMB, WinRM operations

⚠ LSASS does _not_ keep plaintext password.

---

### 🔥 `4 – Batch`

Used for **scheduled tasks** or background jobs.

Use when:  
✔ Task Scheduler jobs  
✔ Scripts executed automatically

Rare in attacks unless weaponizing scheduled tasks persistence.

---

### 🔥 `5 – Service`

Account logs in as a Windows Service.

Use when:  
✔ Running malware or tooling as SYSTEM-equivalent  
✔ Privilege escalation via weak service configs

⚠ Often grants **SeServiceLogonRight**. Useful for persistence.

---

### 🔥 `7 – Unlock`

Session unlock only.  
Almost never used in attacks.

---

### 🔥 `8 – NetworkCleartext`

Credentials **sent in plaintext** if server requests it.

Use in attacks:  
✔ When you want to capture **clear-text password**  
✔ Credential relay / downgrade attacks

⚠ Extremely dangerous if misconfigured — reversible credential exposure.

---

### 🔥 `9 – NewCredentials` _(Most useful for Red Team)_

This is **RUNAS / RunasCs / pass-the-hash style** login.

Creds not used for local logon — only for outbound connections.

Meaning:  
You authenticate as yourself locally but use _other user's credentials_ for network actions.

Use when:  
✔ pivoting inside network  
✔ accessing remote shares/Kerberos with _another identity_  
✔ performing **pass-the-ticket / pass-the-hash / S4U**

Example command:

```powershell
RunasCs.exe target_user Password123! powershell -l 9
```

---

### 🔥 `10 – RemoteInteractive`

This covers RDP & remote GUI sessions.

Same rights as **Interactive**, but remote.

Use when:  
✔ You want full desktop session as user  
✔ You intend to run GUI tools, Mimikatz, BloodHound GUI

⚠ Also stores credentials in LSASS.

---

### 🔥 `11 – CachedInteractive`

Used offline with cached domain credentials.

Use when gaining access to workstation without DC.

Attack implication:  
✔ Extract cached creds via Mimikatz  
✔ Useful in laptop-based engagements

---

---

### 🧠 Summary Table

|ID|Logon Type|Common Use|Credential Exposure|Red Team Use|
|---|---|---|---|---|
|2|Interactive|Local Login|High|Post-exploitation GUI|
|3|Network|SMB/WinRM/Remote Ops|Low|Psexec/WinRM|
|4|Batch|Scheduled Tasks|Medium|Persistence|
|5|Service|Windows Services|Medium–High|Priv Esc / Persistence|
|7|Unlock|Unlock Session|None|Rare|
|8|NetworkCleartext|Sends plaintext credentials|Extreme|Credential theft|
|9|NewCredentials|RUNAS New Token|Low|Pivoting / Kerberos Abuse|
|10|RemoteInteractive|RDP|High|Full interactive shell|
|11|CachedInteractive|Offline Cached Logon|Medium|Lateral movement|

---

> [!Attention]
> i remember using logon session 3 but everytime i try to use rubues and change the tgt and TGS it seems it doesn't use my injected tgt

This is expected behavior if you're using **logon type 3 (Network Logon)** — and it explains exactly why **Rubeus doesn't honor your injected TGT/TGS** even when injection succeeds.

### 💥 Why your injected TGT isn't being used

Logon Type **3 = Network Logon**, and Windows **does NOT create a full logon session with a Kerberos cache** for network logons.

Meaning:

|Feature|Logon Type 2/10 (Interactive/RDP)|Logon Type 3 (Network)|
|---|---|---|
|Kerberos ticket cache stored|✔ Yes|❌ No|
|Accepts injected TGT/TGS|✔ Yes|⚠ Often ignored|
|Kerberos authentication usable|✔ Fully|Only outbound + limited|
|Credential material stored in LSASS|✔ Yes|❌ No (NTLM only)|

So when you do:

```bash
Rubeus.exe ptt /ticket:administrator.kirbi
```

On a **logon type 3 session**, the TGT **injects into memory**, but:

### 👉 It's _not used_ by Kerberos because there's no interactive logon session to attach to.

This is why you can _inject a TGT successfully but can't request TGS or use S4U_.

---

### 🔥 Fix: Use LogonType **2 or 9 or 10** depending on your goal

---

### If you want the TGT to be used normally:

```powershell
RunasCs.exe Administrator Password123! powershell -l 2
```

or for RDP-style interactive session:

```powershell
RunasCs.exe Administrator Password123! powershell -l 10
```

Now `Rubeus ptt` will work exactly as expected.

---

### If you want to pivot silently & still use tickets → **Use LogonType 9 (NewCredentials)**

```powershell
RunasCs.exe Administrator Password123! powershell -l 9
```

Then inject:

```powershell
Rubeus.exe ptt /ticket:admin.kirbi
```

Now you _can request TGS_, do S4U, access CIFS, LDAP, WinRM etc.

⚠ LogonType 9 does not store credentials in LSASS, which is why it’s perfect for red-team ops.

---

### Quick Summary

|Logon Type|Works with injected TGT?|Use When|
|---|---|---|
|**3 (Network)**|❌ Often ignored|Not good for TGT work|
|**2 (Interactive)**|✔ Best for ticket manipulation|Local shell, Mimikatz|
|**10 (RemoteInteractive)**|✔ RDP-context Kerberos|Full desktop access|
|**9 (NewCredentials)**|✔ Perfect for lateral movement|Quiet pivoting + Kerberos abuse|

---
