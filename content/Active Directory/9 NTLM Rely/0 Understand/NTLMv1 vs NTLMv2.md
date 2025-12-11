That is a crucial distinction to understand, as the difference between NTLMv1 and NTLMv2 is central to modern Windows security and the effectiveness of certain hacking techniques.1

The core difference lies in **how the response (the proof of identity) is calculated** and what input factors are used.2 NTLMv2 is a complete overhaul designed to fix the numerous cryptographic weaknesses of NTLMv1.3

Here is a detailed breakdown of the differences and why NTLMv2 is vastly more secure.

---

## 🛡️ NTLMv1 vs. NTLMv2: The Key Differences

|**Feature**|**NTLMv1 (Legacy)**|**NTLMv2 (Current Standard)**|
|---|---|---|
|**Response Algorithm**|Basic DES/MD4-based calculation.|**HMAC-MD5** (much stronger cryptographic hash).|
|**Response Input**|**Password Hash** $\oplus$ **Server Challenge**|**Password Hash** $\oplus$ **Server Challenge** $\oplus$ **Client-Side Data (Timestamp & Client Nonce)**|
|**Response Strength**|**Very Weak.** Highly vulnerable to offline cracking.|**Strong.** Much more resistant to offline cracking.|
|**Replay Attacks**|**Vulnerable.** The response can be replayed if not protected by session signing.|**Mitigated.** The **timestamp** and **Client Nonce** prevent simple replays.|
|**Session Key Generation**|Uses weaker algorithm (56-bit DES).|Uses **HMAC-MD5** for stronger session key generation.|
|**Current Status**|**Deprecated** and disabled by default in modern Windows environments.|**Current, recommended standard** when Kerberos is unavailable.|

---

## 🔒 Why NTLMv2 is VASTLY More Secure

NTLMv2's security comes from two primary changes: a **stronger hashing algorithm** and the inclusion of **time-sensitive client data**.
### 1. The Stronger Hashing Algorithm (HMAC-MD5)4
- **NTLMv1 Weakness:** The response calculation for NTLMv1 is weak and prone to a flaw called **"pass-the-hash"** and **rainbow table attacks**.5 Furthermore, it uses the older, less secure MD4 hash function for the password hash.
- **NTLMv2 Improvement:** NTLMv2 uses **HMAC-MD5 (Hash-based Message Authentication Code)**.6
    - HMAC is a key-dependent hash, meaning the output depends on both the data and a secret key (the password hash).7
    - This makes it significantly harder to reverse-engineer or pre-calculate (like with rainbow tables).
### 2. The Addition of Client-Side Factors (The Nonce and Timestamp)
This is the **most critical security improvement** that makes NTLMv2 highly resistant to basic replay attacks and far harder to crack offline.
In NTLMv2, the client creates a structure called the **"Client Challenge" (or Client Nonce)** which contains unique data that is included in the final response hash.8
#### A. The Client Nonce / Timestamp
- When the client prepares the final response (AUTHENTICATE message), it includes a **current timestamp** and a **random 8-byte value (Client Nonce)** in the response structure.9
- This entire structure, which is unique to that specific authentication request and moment in time, is included in the hash calculation.
#### B. The Hash Calculation (Simplified)
In NTLMv2, the response is not just based on the password hash and the server challenge, but also these new variables:
$$\text{NTLMv2 Response} = \text{HMAC-MD5} (\text{NTLM Hash of Password}, \text{Server Challenge} + \text{Client Nonce/Timestamp})$$
#### C. Impact on Offline Cracking
When an attacker captures an NTLMv2 hash, they now have to crack a hash that is unique not only to the user and the server, but also to the **exact time and random data** of that single authentication event.
- This prevents the attacker from pre-calculating a large table of possible responses.
- The attacker must attempt to guess the password, calculate the complex hash using the captured Server Challenge, Client Nonce, and Timestamp, and compare the result to the captured hash. This adds immense complexity to the cracking process, making NTLMv2 hashes exponentially harder to crack than NTLMv1.
### Summary for the NTLM Relay Module
- **NTLMv1** is essentially dead.10 If you find a system still using it, exploitation is trivial. 
- **NTLMv2** is the standard. Its protection against replay is built-in (due to the timestamp/nonce), but its primary weakness in an authenticated session is _still_ the **lack of SMB Signing**, which is what enables a straight NTLM Relay attack against protocols that don't enforce signing.