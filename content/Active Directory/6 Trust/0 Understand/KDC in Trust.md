
> [!Question] Question
> What about the KDC, do we have different KDC for each subdomain which will serve by different realm? if that the case does that mean when i want to access resource from other domain. I do request TGT ticket from my subdomain then the DC redirect my request with my TGT using ticket to other domain?

Yes 👌 you got it — that’s exactly how Kerberos works across domains in an AD forest. Let’s break it down carefully:
## 1. KDCs per Domain

- Each **domain** in Active Directory has its own **Domain Controllers (DCs)**.    
- Each DC runs a **KDC (Key Distribution Center)** service.
- In Kerberos terms, each domain is its own **realm**.
    - Example:
        - `emea.company.com` = Kerberos realm `EMEA.COMPANY.COM`.
        - `root.company.com` = Kerberos realm `ROOT.COMPANY.COM`.
        - `apac.company.com` = Kerberos realm `APAC.COMPANY.COM`.
So yes — **different subdomains = different KDCs = different Kerberos realms.**

---

## 2. When You Access Resources in Your Own Domain

1. You log on to `emea.company.com`.
2. You get a **TGT** from the **EMEA KDC**.
3. When you access a service in `emea.company.com`, the EMEA KDC gives you a **service ticket** directly.

---

## 3. When You Access Resources in _Another_ Domain

This is where **trusts** + Kerberos **referrals** come in:
1. You log on in `emea.company.com` → get TGT from **EMEA KDC**.
2. You try to access a resource in `apac.company.com`.
3. The EMEA KDC sees:
    - “That service is in another domain I trust.”
    - It gives you a **referral ticket (TGT for the APAC domain KDC)** instead of a direct service ticket.
4. You present this referral TGT to the **APAC KDC**.
5. The APAC KDC issues the actual **service ticket** for the resource in `apac.company.com`.
6. You use that service ticket to talk to the resource server.

---

## 4. Real World Analogy

- Imagine you’re in **EMEA HQ bank** and want to access a service in the **APAC branch**.    
- First, you get a pass (TGT) from EMEA HQ.
- EMEA HQ says: _“Oh, that’s not my branch. Here’s a sealed referral letter for APAC branch.”_
- You take that letter to APAC → they issue you the exact entry ticket to the vault/resource you need.

---

## 5. Across Forests
- Same idea, but now trusts are between **forest roots**.
- If **SID filtering** and Kerberos cross-forest trusts are in place, the referral path goes from your domain → forest root → other forest root → target domain KDC.

---

## TL;DR

- Each domain = its own KDC / Kerberos realm.    
- To access another domain’s resources:
    - Start with your domain TGT.
    - Get a **referral ticket** from your KDC.
    - Use it to request a real service ticket from the other domain’s KDC.

---