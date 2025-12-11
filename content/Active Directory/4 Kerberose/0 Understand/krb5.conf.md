
```txt

[libdefaults]
        default_realm = DUMMY.REALM
        dns_lookup_kdc = true
        dns_lookup_realm = true

[realms]
        DUMMY.REALM = {
                kdc = {{dc_hostname}}.{{realm_placeholder}}
                admin_server = {{dc_hostname}}.{{realm_placeholder}}
                default_domain = {{realm_placeholder}}
        }

[domain_realm]
        .{{realm_placeholder}} = {{realm_placehoder}}
        {{realm_placeholder}} = {{realm_placehoder}}


____________

[libdefaults]
    default_realm = FLUFFY.HTB
    dns_lookup_kdc = false
    dns_lookup_realm = false
    ticket_lifetime = 24h
    forwardable = true

[realms]
    FLUFFY.HTB = {
        kdc = dc01.fluffy.htb
        admin_server = dc01.fluffy.htb
    }
    LOGISTICS.INLANEFREIGHT.LOCAL = {
        kdc = academy-ea-dc02.inlanefreight.local
        admin_server = academy-ea-dc02.inlanefreight.local
    }

[domain_realm]
    .fluffy.htb = FLUFFY.HTB
    fluffy.htb = FLUFFY.HTB
    .logistics.inlanefreight.local = LOGISTICS.INLANEFREIGHT.LOCAL
    logistics.inlanefreight.local = LOGISTICS.INLANEFREIGHT.LOCAL



```


```sh
faketime "$( ntpdate -q voleur.htb | cut -d ' ' -f 1,2)" evil-winrm -i dc.voleur.htb -k -u svc_winrm -r voleur.htb
```