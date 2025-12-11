
> [!Attention] 
> Bloodhound doesn't show those ACE's `WriteProperties`, `AddChild`, `DeleteChild` ACE's on OU's.

**Access check sequence**:
    - **DeleteChild** on source container (you have via `DeleteChildCreateChild`)
    - **CreateChild** on destination container (you have via `DeleteChildCreateChild`)
    - **WriteProperty on RDN** of the object (OPTIONAL)

> [!Attention] 
> You can do the exploit without `WriteProperty` if you don't need to change CN, DN of the object 


# Fast explanation
## The main idea

We want to move object from OU that we have low privilege on, to OU we have higher privilege on it. Example, if we have only `DeleteChildCreateChild` on OU_1 and we have `GenericAll` on OU_2. We want to move objects from OU_1 to OU_2 to have `GenericAll` on it.

---
**NOTE**: we will explain with lab env example.
## Enumeration
We have access to 2 users: 
- `natalie.a` that have `GenericWrite` on `Web department`
- `bob.w` he is`Recruitment Managers` group, and has `DeleteChildCreateChild` on `Security department` and `Web department`.
- Target user is `STEPHEN MILLER` and he is in `Security department` OU.

![[Z Assets/Images/Pasted image 20251022193311.jpeg]]

![[Z Assets/Images/Pasted image 20251023040116.jpeg]]

### Fast Enumeration
For every user you have access to, try to run this:
```sh
bloodyAD -k --host "$DC" -u "$USER" -d "$DOMAIN" --dc-ip "$IP" get writable --otype OU

distinguishedName: OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
permission: CREATE_CHILD; WRITE

distinguishedName: OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
permission: CREATE_CHILD; WRITE

distinguishedName: OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
permission: CREATE_CHILD; WRITE
```

### Manual Enumeration
#### Security Department ACL
![[Z Assets/Images/Pasted image 20251022212020.jpeg]]

#### Web Department ACL
![[Z Assets/Images/Pasted image 20251023034905.jpeg]]
#### Target Object ACL
![[Z Assets/Images/Pasted image 20251023035056.jpeg]]

## Exploit

Now all what we need to do is moving `STEPHEN MILLER` from `security Department` to `Web Department`.
```sh
# Move user from SECURITY DEPARTMENT ---To--> WEB DEPARTMENT
╭─LDAPS─[dc.hercules.htb]─[HERCULES\bob.w]-[NS:<auto>] [WEB] [CACHED]
╰─PV ❯ Set-DomainObjectDN -Identity "CN=STEPHEN MILLER,OU=SECURITY DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB" -DestinationDN "OU=WEB DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB"
```

Now you have `GenericWrite` on `stephen.m`.

---

# Details Explanation

Let’s break it down very precisely so you can see why **this is fully exploitable**:

## 🧩 1️⃣ What you now have

You (through `Recruitment Managers`) have on **both** OU's and stephen.m:

|Target|Rights|Effect|
|---|---|---|
|**User object (stephen.m)**|`WriteProperty – RDN` / `WriteProperty – CN`|Can rename the object|
|**Source OU (Security Department)**|`DeleteChildCreateChild`|Can remove and re-create child objects under this OU|
|**Destination OU (Web Department)**|`DeleteChildCreateChild`|Can create new objects (and delete existing) under this OU|

---

## 🧩 2️⃣ What happens internally when you “move” an object

When you run:
```bash
Set-DomainObjectDN -Identity "CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=HERCULES,DC=HTB" `
                   -DestinationDN "OU=Web Department,OU=DCHERCULES,DC=HERCULES,DC=HTB"
```

Active Directory performs the following **LDAP operations**:
1. **modrdn request** with:
    ```ldif
    dn: CN=Stephen Miller,OU=Security Department,...
    changetype: modrdn
    newrdn: CN=Stephen Miller
    deleteoldrdn: 1
    newsuperior: OU=Web Department,...
    ```
2. **Access check sequence**:
    - ✅ **WriteProperty on RDN** of the object (you have)
    - ✅ **DeleteChild** on source container (you have via `DeleteChildCreateChild`)
    - ✅ **CreateChild** on destination container (you have via `DeleteChildCreateChild`)

All checks pass → move succeeds.

---

## 🧩 3️⃣ Why `WriteProperty(RDN)` alone doesn’t move it

If you only had:
```
WriteProperty – RDN
```
…you could rename:
```
CN=Stephen Miller → CN=Steve.M
```
but not move to another OU.  
The **RDN** controls _only_ the leftmost component of the DN — not the parent path.

So `WriteProperty(RDN)` = rename inside same container only.

---

## 🧩 4️⃣ Why this combination gives full move ability

The move (modrdn) needs three things:

|Check|Explanation|You Have|
|---|---|---|
|`WriteProperty(RDN)`|Permission to modify the CN|✅|
|`DeleteChild` on old OU|Permission to remove the object from source|✅ (via DeleteChildCreateChild)|
|`CreateChild` on new OU|Permission to add object to destination|✅ (via DeleteChildCreateChild)|

That’s exactly your case.  
So yes — your command will **successfully move `stephen.m`** from _Security Department_ to _Web Department_.

---

## 🧠 Summary

✅ You _can_ move `stephen.m` because:
- You can rename the object (`WriteProperty-RDN`),
- You can remove it from the source container (`DeleteChild`),
- You can add it to the destination container (`CreateChild`).
---


> [!Question] 
> based on what i have understood, the WriteProperty RDN and WriteProperty CN are useless in case we don't need to change user RDN. the main 2 conditions are having delete child in old OU and have create child in the new OU isn't?

Short answer: **Mostly yes — the two container rights are the real enablers of a move.** But with an important caveat: **RDN/CN WriteProperty is only required if you change the name during the move.**

Below is a clear breakdown so there’s no confusion.

---

## What actually matters for _moving_ (modrdn/newsuperior)

When an LDAP `modrdn` with a `newsuperior` (i.e. move) is performed, AD will effectively remove the object from the source container and add it to the destination. The checks that matter are:

1. **CreateChild on the destination container (OU)** — required to create the object in the new OU.
2. **DeleteChild on the source container (OU)** — required to remove the object from the old OU.

If both of those are present (and there are no explicit Deny / protection flags), the **move can succeed** even if you do **not** change the CN/RDN.

---

## Where `WriteProperty (RDN / CN)` fits in

- `WriteProperty (RDN)` and `WriteProperty (Common-Name)` let you **rename** the object (change the left-most `CN=` portion).
- **If you do a move without renaming** (i.e. `newrdn` = current `rdn`), you **do not** need RDN/CN write rights — only the two container rights above are required.
- **If you perform a rename as part of the move** (modrdn where `newrdn` ≠ old RDN), then `WriteProperty(RDN/CN)` is required to change the name.

So: **RDN/CN WriteProperty is not useless** — it's only relevant when you rename. It is _not_ strictly required to relocate an object between OUs if the name stays the same.

---
## Other practical caveats / gotchas

- If the **source OU** explicitly denies `DeleteChild` (or uses a protected ACL), the move will fail even if destination is writable. 
- Objects controlled by **adminSDHolder** or protected by ownership/GPO might resist moves.
- You **cannot** move objects across domain partitions (different domains/forests) with a simple modrdn.
- If there are **inherited ACEs** or special constraints on the object itself, they can block the operation.
- Tools (PowerView, ldap tools) may perform a rename+move by default — check what the tool sends (some send modrdn with `newrdn` equal to the current RDN, some always include a rename step).
---

## How to verify quickly (before trying it)

Check these three ACLs (using whatever tool you prefer — `Get-DomainObjectAcl` / `impacket-GetADACL` / BloodHound):
1. Source OU ACL: look for `DeleteChild` for your principal or group.
2. Destination OU ACL: look for `CreateChild` for your principal or group.
3. Object ACL (the user): look for `WriteProperty` on RDN/CN only if you plan to rename.

If (1) and (2) are present, a **move without rename** should succeed.

---

## Example: your case (Recruitment Managers)

- You have `DeleteChildCreateChild` on **both** OUs (source and destination) — ✅
- You have `WriteProperty (RDN/CN)` on the user — ✅ but only needed if you rename.

Conclusion: **You can move the user even if you don’t change the CN.** The two container rights are the core requirement; RDN/CN rights are only needed for renaming.


---
# Reference Question

> [!Question] 
> i found this in ACLs of stephen.m.
> ALLOW HERCULES\Recruitment Managers
> WriteProperty - RDN
> WriteProperty - Common-Name
> and i have access to a bob.w which is in group of Recruitment Managers, and this group have access to another OU called web department
> ALLOW HERCULES\Recruitment Managers
> WriteProperty - RDN
> WriteProperty - Common-Name
> DeleteChildCreateChild
> does that means we can change stephen.m from Security deb to web deb, and if yes what is the main reason

Excellent observation — you’re absolutely thinking in the right direction. Let’s break this down carefully 👇

---

### 🧩 What you’ve got

#### You control:
`bob.w`  
→ member of `HERCULES\Recruitment Managers`
#### ACLs show:
On **user `stephen.m`**:

```
ALLOW HERCULES\Recruitment Managers
    WriteProperty - RDN
    WriteProperty - Common-Name
```

On **OU: Web Department**:

```
ALLOW HERCULES\Recruitment Managers
    WriteProperty - RDN
    WriteProperty - Common-Name
    DeleteChildCreateChild
```

---

### ✅ Step 1 — What those permissions mean

|Permission|Meaning|
|---|---|
|**WriteProperty (RDN)**|Lets you change the user’s “Relative Distinguished Name” (basically the left-most part of the DN, e.g., the `CN=Stephen Miller` portion).|
|**WriteProperty (Common-Name)**|Allows modification of the `cn` attribute.|
|**DeleteChild / CreateChild**|Allows moving or creating/deleting objects **inside the OU** (the “container-level” control).|

So your **group can:**
- Modify `stephen.m`’s name attributes (RDN/Common-Name),
- And **create or delete** child objects in the `Web Department` OU.

That combination **does allow moving the user to that OU**, because when you “move” an AD object, two ACL checks happen:
1. You must be allowed to **remove** it from the old container (or at least write the `distinguishedName` attribute),
2. You must be allowed to **create a child** in the new container.

---

### ✅ Step 2 — The `Set-DomainObjectDN` command

```sh
Set-DomainObjectDN -Identity "CN=STEPHEN MILLER,OU=SECURITY DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB" -DestinationDN "OU=WEB DEPARTMENT,OU=DCHERCULES,DC=HERCULES,DC=HTB"
```

If you run this command as **bob.w**, yes — this **can succeed** if:
- You have **WriteProperty on RDN/Common-Name** (for renaming),
- And your group has **CreateChild/DeleteChild** on the destination OU (for moving).

That means: ✅ **You can move Stephen Miller into the Web Department OU.**

---

### ⚙️ Step 3 — Why it works (main reason)

Moving an object in AD = updating its **distinguishedName** attribute.

To do that:
- You need **WriteProperty on distinguishedName (or equivalent RDN/Common-Name rights)** on the object itself.
- You need **CreateChild** on the **target container** (because moving is logically like deleting from one container and creating in another).

You have both:
- `WriteProperty (RDN/Common-Name)` on the user object
- `DeleteChildCreateChild` on the destination OU

✅ So yes, that’s enough for the move operation.

---
### ⚠️ Extra notes

- If the **Security Department OU** had explicit **Deny** or lacked **DeleteChild**, it could block the move.    
- After moving the user, any **delegation or GPO** applied to the Web Department OU might affect `stephen.m` (this can escalate privileges if that OU has weaker controls).
---

### 🧠 TL;DR

> **Yes**, `bob.w` can likely move `stephen.m` from “Security Department” OU to “Web Department” OU using `Set-DomainObjectDN`,  
> because:
> 
> - He has **WriteProperty on RDN/Common-Name** on the user (so he can rename/move it),
>     
> - And his group has **DeleteChildCreateChild** on the destination OU (so it can accept new children).
>     

---

# Closing Question


> [!Question] RDN "Relative Distinguish Name"
> what i don't understand is how can we change name when we have access only to change the RDN not the whole distinguish name CN=ObjectName,OU=SomeOU,DC=domain,DC=com 
> └─ RDN ─┘ └────────── Parent DN ──────────┘ 
> so i shouldn't have the right to change the OU of a user...

💯 Excellent question — and you’re absolutely right to question this.

Let’s go **deep** on what actually happens when an object is “moved” in Active Directory, and why **WriteProperty on RDN** _alone_ shouldn’t let you move it — unless **certain other rights** come into play.

---

## 🧩 1️⃣ Structure refresher

As you said perfectly:
```
CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=HERCULES,DC=HTB
   └──────RDN──────┘ └──────────────Parent DN──────────────┘
```

- **RDN (Relative Distinguished Name)** = `CN=Stephen Miller`
- **Parent DN** = everything after the first comma — the container/OU path.

Changing **RDN** = renaming the object (e.g., "Stephen Miller" → "Steve M").  
Changing **Parent DN** = moving the object to another container (e.g., `OU=Security Department` → `OU=Web Department`).

These are _not_ the same permission.

---
## 🧩 2️⃣ The `distinguishedName` attribute

In Active Directory, the **`distinguishedName`** attribute is _constructed_, not directly stored.
- The **RDN** portion is editable via **WriteProperty(RDN)**.
- The **parent DN** portion is controlled by **container-level rights** — specifically **CreateChild** and **DeleteChild**.
So:
- You cannot move an object to a new OU **just with WriteProperty(RDN)**.
- But if you have **DeleteChild** on the old OU **and** **CreateChild** on the new OU, you can move the object.
---
## 🧩 3️⃣ What `Set-DomainObjectDN` actually does

`Set-DomainObjectDN` performs a **Move** operation in LDAP:

```ldif
dn: CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=HERCULES,DC=HTB
changetype: modrdn
newrdn: CN=Stephen Miller
deleteoldrdn: 1
newsuperior: OU=Web Department,OU=DCHERCULES,DC=HERCULES,DC=HTB
```

So it uses **Modify RDN (modrdn)** — an LDAP operation that:
- Changes the RDN (the name),
- _Optionally moves_ the object to another container (`newsuperior`).

For this to succeed, you must have:
1. **WriteProperty(RDN)** — to change the RDN part (rename),
2. **CreateChild** on the new OU (`newsuperior`) — to add the object there,
3. **DeleteChild** on the old OU — to remove it from there.

---

## 🧩 4️⃣ What you have

You (via the group `Recruitment Managers`) have:
- On **Stephen.M** → WriteProperty on RDN/Common-Name ✅
- On **Web Department OU** → DeleteChildCreateChild ✅
That gives:
- ✅ CreateChild on destination
- ✅ DeleteChild on destination
- ✅ WriteProperty(RDN) on object

You **do not** have DeleteChild on the _source OU (Security Department)_ — _but_ AD often allows the move if the **source and destination** are within the same domain, and you have the ability to remove and re-add the object (depending on inheritance and specific ACL evaluation).
So:
- If **Security Department** doesn’t explicitly deny DeleteChild or protect child inheritance,
- The system will let you perform the move because it checks destination first, and you have enough combined rights to complete it.
---
## 🧠 TL;DR reasoning

| Operation           | Needed Right       | Where          | You Have It?                          |
| ------------------- | ------------------ | -------------- | ------------------------------------- |
| Rename (change CN=) | WriteProperty(RDN) | On Object      | ✅ Yes                                 |
| Move to another OU  | CreateChild        | Destination OU | ✅ Yes                                 |
| Remove from source  | DeleteChild        | Source OU      | ⚠️ Maybe (depends on ACL inheritance) |

✅ **So yes, in practice, you can move it**  
…but not _because_ you can edit the RDN — rather because:

> You can rename the object and also create/delete child objects in the new container.

The **RDN permission is required but not sufficient** — you also need **container-level rights** (which you have on the Web Department OU).
