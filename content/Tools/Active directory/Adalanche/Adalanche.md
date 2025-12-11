The binaries/ folder will contain cross-platform binaries of Adalanche ; in the case of Pwnbox , we will use the adalanche-linux-x64-* one.
## Data Collection
To collect data from domain controllers and domain-joined Windows systems, we will use the collect activedirectory command, passing the domain name for the --domain option, the domain controller's IP address for the --server option, and the username and password of the domain user for the --username and --password options, respectively:
```sh
./adalanche-linux-x64-v2024.1.11-43-g7774681 collect activedirectory --domain inlanefreight.local --server 10.129.229.224 --username 'david' --password 'SecurePassDav!d5'

#kerberos auth
~/.opt/windows/Adalanche/binaries/adalanche-linux-x64-latestrelease-112-g69eadcf collect activedirectory --domain "$DOMAIN" --server "$DC" --username "$USER" --authmode kerberoscache
```

## Data Analysis
Once Adalanche finishes collecting data, we will use the analyze command to launch Adalanche 's interactive discovery tool, passing the directory of the collected data for the 
`--datapath` option:
```sh
./adalanche-linux-x64-v2024.1.11-44-gf1573f2 analyze --datapath data
#http://127.0.0.1:8080
```
