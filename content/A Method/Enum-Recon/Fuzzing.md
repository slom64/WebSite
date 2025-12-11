When use ffuf make sure to put right content-type, because even if you have right data but wrong content-type this will lead to unexpected output:
```sh
ffuf -w /opt/lists/rockyou.txt -u http://lookup.thm/login.php -X POST -d 'username=jose&password=FUZZ' -ic -ac -H 'Content-Type: application/x-www-form-urlencoded'
```