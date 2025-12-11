Virtual host
```sh
gobuster vhost -u https://example.com -t 50 -w /wordlists/Discovery/DNS/subdomains-top1million-5000.txt
```



DNS enumeration
```sh
gobuster dns -q -r 8.8.8.8 -d example.com -w wordlists/Discovery/DNS/subdomains-top1million-5000.txt -t 4 --delay 1s -o results.txt"
```

| Option    | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `-q`      | Don't print the banner and other noise                       |
| `-r`      | Use custom DNS server (format server.com or server.com:port) |
| `-d`      | Domain String                                                |
| `-w`      | wordlist path                                                |
| `-t`      | threads                                                      |
| `--delay` | delay duration                                               |
| `-o`      | output file (default to stdout)                              |
