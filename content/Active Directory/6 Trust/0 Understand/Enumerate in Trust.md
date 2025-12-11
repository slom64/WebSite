When i am at subdomain and the parent domain trust me, can i enumerate the parent AD objects and other subdoamin AD objects?

---
## 1. Trust Direction Refresher

- **Parent trusts Child** → means **child users** can access parent resources.
- **Child trusts Parent** → means **parent users** can access child resources.
- Trusts are usually **two-way and transitive inside a forest**, so:
    - Parent ↔ Child
    - All children ↔ each other through the parent.

---
## 2. What Can You See From a Child Domain?

If you’ve compromised an account in **ChildDomain.Company.com**:
- You **can authenticate against the Parent domain DC** (since the trust is two-way by default in a forest).
- You can query LDAP on the **parent DC** using your child creds.
- But: what you can _actually read_ depends on ACLs (permissions on objects).

By default in AD:

- **Authenticated Users** (any valid user in the forest) can read a lot of directory info → domain/forest structure, list of objects, group memberships, etc.
- That means **yes, from a child domain you can enumerate the parent domain and other child domains** (to a large extent).

---

## 3. Enumerating Other Child Domains

Since trusts in a forest are **transitive**, if Parent trusts ChildA and Parent trusts ChildB:
- ChildA users are also implicitly trusted by ChildB.
- So from ChildA, you can usually **enumerate objects in ChildB** as well.

Again, ACLs apply, but by default you’ll see plenty (like user/group objects).

---

## 4. Real-World Example

Let’s say we have this forest:

- `root.company.com`    
- `emea.company.com` (child)
- `apac.company.com` (child)

You compromise a normal user in `emea.company.com`.

- You can bind with LDAP to `root.company.com` DC and enumerate users/groups.
- You can also bind to `apac.company.com` DC (because of transitive trust) and enumerate there.
- You might not be able to **access sensitive resources** (like SYSVOL files protected by ACLs), but you can still map the structure of the whole forest

---
## 5. Why This Matters

- From **any foothold in the forest**, attackers can usually enumerate **the whole forest structure**.
- This is why tools like **BloodHound** or **AD Explorer** work so well from a single compromised account.    
---

**Answer in one line:**  
Yes — if the parent domain trusts you (normal inside a forest), you can usually enumerate AD objects in the parent and sibling child domains. You won’t get _everything_, but you’ll see enough to map the forest.

---

Do you want me to also explain **what extra you can see if you are “Enterprise Admin” vs just Authenticated User** across domains?