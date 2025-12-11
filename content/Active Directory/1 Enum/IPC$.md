if IPC$ is readable using anonymous login "NULL session", then you can do rid-bruteforcing  to get  usernames in the doamin:
```sh
nxc smb SOUPEDECODE.LOCAL -u 'guest' -p '' --rid-brute 
grep 'SOUPEDECODE\\' rid_brute.txt | cut -d':' -f2- | sed -E 's/.*SOUPEDECODE\\(.*) \(SidType.*/\1/' | grep -v '\$' > usernames.txt
```