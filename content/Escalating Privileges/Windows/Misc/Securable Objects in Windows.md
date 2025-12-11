Yes — Windows has **many** securable objects beyond files, registry keys, services, and AD objects.  
Anything that has an ACL (Access Control List) is a **Windows securable object**.

Here is the **complete list that matters for privilege escalation, red teaming, forensics, and OS internals**.

---

# ✅ **1. Files & Directories (NTFS)**

Securable? ✔  
Stored in: NTFS ACL  
Examples:
- Executables
- DLLs
- Config files
- User directories
---
# ✅ **2. Registry Keys**
Securable? ✔  
Stored in: Registry ACL  
Examples:
- HKLM\Software\…
- HKLM\System\CurrentControlSet\Services*
Registry entries themselves (the values) don't have ACLs — only **keys**.

---
# ✅ **3. Services (SCM objects)**

Securable? ✔  
Stored in: SCM security descriptors  
Examples:
- Service start/stop permission
- Change config permission
- Service triggers
---
# 🟦 **4. Active Directory Objects**
Securable? ✔  
Stored in: AD ACL  
Examples:
- Users
- Groups
- Computers
- OUs
- GPOs
- Service accounts (gMSA, MSA)
---
# 🟩 **5. WMI Namespaces**

Securable? ✔  
Stored in: CIM repository  
Examples:
- `root\cimv2`
- `root\subscription`
- `root\default`
Attack vector
- WMI permanent event subscription persistence
- Unprivileged WMI execution
---
# 🟨 **6. COM Objects (DCOM)**
Securable? ✔  
Stored in: COM ACLs  
Examples:
- Excel.Application
- ShellWindows
- MMC20.Application
Attack vector:
- COM hijacking
- High-integrity COM object abuse
- Activating elevated COM objects
---
# 🟧 **7. Named Pipes**
Securable? ✔  
Stored in: Pipe ACLs  
Examples:
- `\\.\pipe\spoolss`
- `\\.\pipe\winlogon`
Used in:
- LRPC (Local RPC)
- SMB named pipes
- Privilege escalation via impersonation
---
# 🟥 **8. Windows Objects (Kernel Objects)**
Securable? ✔  
Stored in: Object Manager  
Examples:
- Mutexes
- Events
- Semaphores
- Sections (shared memory)
- Job objects
- Symbolic links
Attack vector:
- Handle inheritance attacks
- Potatoes (RottenPotato, PrintSpoofer, etc.)
---
# 🟦 **9. Scheduled Tasks**
Securable? ✔  
Stored in: Task Scheduler  
Examples:
- Running tasks
- Editing task XML
- Changing the action image path
Attack vector:
- Unquoted task path
- Modify task to run malicious binary
---
# 🟩 **10. Firewall Rules**
Securable? ✔  
Stored in: Firewall policy store  
Examples:
- Add/remove inbound rules
- Modify existing rules
---
# 🟪 **11. Event Logs**
Securable? ✔  
Stored in: Event log channels  
Example:
- `Microsoft-Windows-TaskScheduler/Operational`
- Security logs
Permissions determine who can read/clear/write
---
# 🟫 **12. Shared Folders / SMB Shares**

Securable? ✔  
Two permission layers:
- Share permissions (SMB)
- NTFS permissions
Attack vector:
- Write access to a share containing executables
- Drop malicious DLLs
---
# ⬛ **13. Printer Objects (Spooler)**
Securable? ✔  
Stored in: Spooler ACL  
Example:
- Who can install drivers
- Who can load plugins
Attack vector:
- PrintNightmare
- Malicious drivers    
---
# 🟫 **14. Network Resources**
Securable? ✔  
Examples:
- Named endpoints
- Pipe-based RPC servers
- WinRM access restrictions
---
# 🟩 **15. Windows App Packages (UWP)**
Securable? ✔  
Examples:
- Capabilities
- Package manifest permissions
---
# 🔵 Summary — Windows Securable Objects

Here’s the complete list in one place:

| Category             | Examples                             |
| -------------------- | ------------------------------------ |
| **File System**      | Files, directories                   |
| **Registry**         | Keys                                 |
| **Services (SCM)**   | Service control ACLs                 |
| **Active Directory** | Users, OUs, GPOs, computers          |
| **WMI**              | Namespaces, event filters            |
| **COM / DCOM**       | CLSIDs, AppIDs                       |
| **Kernel Objects**   | Mutex, events, pipes, sections, jobs |
| **Named Pipes**      | SMB pipes, RPC endpoints             |
| **Scheduled Tasks**  | Task definitions                     |
| **Firewall Policy**  | Firewall rules                       |
| **Event Logs**       | Log channels                         |
| **SMB Shares**       | Share ACLs                           |
| **Printer Objects**  | Print drivers, printers              |
| **UWP Packages**     | App capabilities                     |

---

# If you want

I can explain **for each one**:

- How to list permissions
    
- How attackers abuse them
    
- How to protect them
    

Just tell me which category you want to deep dive into.