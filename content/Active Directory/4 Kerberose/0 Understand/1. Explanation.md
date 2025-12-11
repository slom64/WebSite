
## KDC

It supplies tickets and generate temporary session keys. 
KDC Stores all symmetric keys for both users and services.

It contain 2 servers inside KDC: ==authentication server== `AS` , ==ticket granting server== `TGS`

### Authentication server
confirms that known user is making access request and issue `TGT` .

### Ticket Granting Server
Confirms that a user is making an access reques to known service, and issue `ST` service tickets. `TGS` 


![[Z Assets/Images/Pasted image 20250817212426.png]]

Before start, there is 2 types of messages:
- `authenticators`: used to authenticate the user to KDC, and to authenticate service to user `"mutual authentication"`.
- `tickets`


> [!NOTE] 
> When we say TGS that doesn't mean we refer to TGS ticket in this context. but we refer to "**Ticket Granting Service**" 


# Steps
Mainly there is 3 important phases sent from user:

## phase 1
- user send `unencrypted` message to `authentication service`, contain `userID` and `serviceID`
- `Authentication service` looks at `userID`, Authentication service has list of all users and their sercert keys. So, it check if userID in the list, if so it graps the `user secret key`.
- Authentication service create 2 messages then send them to user:
	- First message contain `TGSName/ID`, timestamp, lifetime, and `TGS session key`.                                                                                               🔒All encrypted using `user secret key`.
	- Second message `TGT` *ticket granting ticket* contain `UserID`, `TGSName/ID`, TimeStamp, UserIP, TGT lifetime, `TGS session key`.    🔒All ecrypted using `TGS secret key`
- The user decrypt the first message using `user secret key`, 
	- `user secret key` is generated using user password, added to it username@realm.com as salt. then we get the hash. This hash is the 🔓 `user secret key`.
	- This is the first step of validating users password, because if we have wrong user password, we won't be able to decrypt the message.
	- Now we have `TGS session key`, which will be very helpful.

> [!NOTE] **Ticket granting ticket decryption**
> **User can't decrypt TGT *'ticket granting ticket'* because he doesn't has TGS secret key**.

![[Z Assets/Images/Pasted image 20250918182530.jpeg]]

![[Z Assets/Images/Pasted image 20250918182602.jpeg]]

## phase 2
- user send 3 messages:
	- first message:`TGT`                                                                                                               🔒ecrypted using `TGS secret key`
	- second message contain `ServiceName/ID`, Requested lifetime of ticket.      🔓Unecrypted
	- Third message:`User authenticator`, which contain `UserID`, timestamp.   🔒encrypted using `TGS session key`
		- Doing this, we make sure no one has intercepted TGT on the wire then relay it again, because if someone did it they won't know the `TGS session key`.
- `Ticket granting server` starts by looking at `serviceName/ID` in the unecrypted message, if its not found the request is denyed. if found then `TGS` grap `service secret key`.
- Then 🔓 decrypte `TGT` using `TGS secret key`, So we get `TGS session key` that will be used to 🔓decrypte `User authenticator`.
- Now everything is unencrypted. The TGS start to validate data:
	- check if `UserID` in `TGT` and `User authenticator` match each other. And TimeStamp is checked too.
	- Compare the IP address in TGT with the IP that is currently requesting.
	- check TGT has not expired.
- If all is good, then `TGS` use `TGS cache` to cache recently used `User authenticator`. And TGS make sure `User authenticator` isn't already in the cache. protection from replay.
- The TGS starts to create 2 messages and create random symmetric `service session key`:
	- First message: `ServiceID/Name` that the user want, TimeStamp, Lifetime, `service session key`.                                                                        🔒ecrypted using `TGS session key`
	- second message is`ST` `'service ticket'`: it contain `userID`, `ServiceID/Name`, timestamp, User IP, lifetime, `service session key`. 🔒encrypted by`service secret key`
- User 🔓decrypte the first message using `TGS session key`
	- now user has a copy of `serive session key`.
	- User can't decrypte `ST`, because he doesn't has `service secret key`.

![[Z Assets/Images/Pasted image 20250918182645.jpeg]]

![[Z Assets/Images/Pasted image 20250918182728.jpeg]]


## phase 3
- user send 2 messages:
	- first message: `ST` `'service ticket'`                                           🔒ecrypted using `service private key`
	- second message `User authenticator`: UserID, TimeStamp  🔒ecrypted using `service session key`
- Now the service 🔓decrypte the `ST` using its `service private key`, now it has `service session key`. Then it 🔓decrypte `User authenticator` using `service session key`.
- Then service validate the data is identical.
	- Compare `UserID`, timeStamp in `user authenticator` and `ST`
	- Compare `IP address` of requester and `IP address` inside `ST`
	- Check service ticket isn't expired.
- Then the service use it cache to save recently `User authenticator`, and make sure there is no other entry for the same user authenticator. replay protaction
- Then the service send it own authenticator `Service authenticator`: contain `ServiceName/ID`, timeStamp. 🔒ecrypted using `service session key`
- User decrypte `Service authenticator` using `service session key`. And user will cache the encrypted `ST` for future use.
![[Z Assets/Images/Pasted image 20250918182832.jpeg]]

![[Z Assets/Images/Pasted image 20250918182914.jpeg]]

> [!question] TGT request cracking
> I have questions in active directoy kerberos, we at first send req with userid and serviceid and other things, then tgs respond with tgt, and (tgs name, tgs session key) encrypted with client secret key. Thats weird because we already have the tgt and we didn’t use our password to get it. Now we can use it, there is something missing that would be too easy for attackers to request all users tgt.
You've hit on one of the most important security concepts in Kerberos! You're absolutely right to be suspicious - if anyone could just request a TGT for any user without authentication, that would be a massive security hole.

Here's what you're missing: **The initial request (AS-REQ) doesn't just contain the user ID. It contains proof that the client knows the user's password.**
## The Key Detail: Timestamp Encryption
When a client sends the initial Authentication Server Request (AS-REQ), it includes:
1. **User principal name** (e.g., user@DOMAIN.COM)
2. **Service principal name** (krbtgt/DOMAIN.COM)
3. **A pre-authentication timestamp** - encrypted with the user's password-derived key
## The Pre-Authentication Process
Here's the crucial step you're missing:
1. **Client generates a timestamp** (current time)
2. **Client encrypts the timestamp** using a cryptographic key derived from the user's password
3. **Client sends the encrypted timestamp** in the AS-REQ

The Authentication Server (AS):
1. Looks up the user's account and retrieves the stored password hash
2. Uses this to derive the same cryptographic key
3. **Attempts to decrypt the timestamp**
4. If decryption succeeds and the timestamp is recent (prevents replay attacks), authentication is successful
## Why This is Secure
- **Without the password**, an attacker cannot create a valid encrypted timestamp
- The KDC won't issue a TGT without successful pre-authentication
- **Pre-authentication is enabled by default** in modern Windows domains
## What If Pre-Auth is Disabled?
In rare cases where pre-authentication is disabled (not recommended!), you're right - an attacker could request a TGT for any user without knowing their password. This is why Microsoft enables pre-authentication by default and considers disabling it a security risk.

## The Complete Secure Flow:
1. **AS-REQ**: UserID + encrypted timestamp (proof of password knowledge)
2. **AS-REP**: TGT encrypted with KRBTGT key + session key encrypted with user's key
3. **Only someone with the password** can decrypt the session key to use the TGT

So you're absolutely right to question this - without pre-authentication, it would be vulnerable. But the pre-authentication step is what makes the initial exchange secure.

---

> [!question] TGT cracking
> ahh i see, so if pre-authentication is enabled, we use the user key to encrypt the timestamp and that works as proof of our identity. but if its disabled then we can request any user tgt then subject it to offline cracking.
### Quick confirmation
- When **pre-auth is enabled**, the client proves knowledge of the password by sending a **PA-ENC-TIMESTAMP** (or using PKINIT for certs) encrypted with the user’s long-term key (derived from their password). The KDC only responds when that proof is valid. That prevents an attacker from getting a usable, crackable blob.
- When **pre-auth is disabled**, the KDC will reply to a plain AS-REQ with an **AS-REP** that contains the encrypted EncPart (the session key + related fields) encrypted with the user’s long-term key. An attacker can collect that blob offline and brute-force it to recover the user password or hash. This is **AS-REP Roasting** — you’re not cracking the TGT itself (the TGT is always encrypted to `krbtgt`), you’re cracking the part encrypted with the user key inside the AS-REP.
### How the offline attack flows (high level)
1. Attacker requests AS-REP for target username (no preauth required).
2. KDC returns AS-REP containing EncPart encrypted under the user key.
3. Attacker runs offline cracking (dictionary/wordlist, hashcat/john/etc.) against that encrypted blob until they recover the password/hash.
4. With the password/hash they can decrypt the EncPart, get the TGS session key, and then use the TGT legitimately.

---

> [!question] TGT rely
> If we were able to get TGT by doing MITM attack, that won't be enough to use the TGT because we need the session key, is that right?
