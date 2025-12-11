## What do they mean by _No authorized signatures are required_?

This refers to a feature in AD CS templates called **Authorized Signatures** (or “issuance requirements”).
- A template can be configured so that **one or more additional signatures** (from another cert) are required to issue a cert.
- Example: “To issue this cert, it must be co-signed by a manager’s cert or security officer’s cert.”
- If “No authorized signatures are required,” then you (as requester) can directly get the cert without needing someone else’s approval.

So in ESC3 write-ups, they emphasize that **if authorized signatures were required, abuse would be much harder** — but misconfigured templates often leave this off.