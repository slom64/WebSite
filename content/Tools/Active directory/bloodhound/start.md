
The command executed Bloodhound.py with the user `forend`. We specified our nameserver as the Domain Controller with the `-ns` flag and the domain, INLANEFREIGHt.LOCAL with the `-d` flag. The `-c all` flag told the tool to run all checks. Once the script finishes, we will see the output files in the current working directory in the format of <date_object.json>.
```shell

bloodhound-python -u 'levi.james' -p 'KingofAkron2025!'  -d puppy.htb -ns 10.xx.xx.xx -c All --zip
bloodhound-ce-python -d fluffy.htb  -dc dc01.fluffy.htb -u 'j.fleischman' -k -no-pass -ns 10.10.11.69
```

We can either upload each JSON file one by one or zip them first with a command such as `zip -r ilfreight_bh.zip *.json` and upload the Zip file. We do this by clicking the `Upload Data` button on the right side of the window (green arrow). When the file browser window pops up to select a file, choose the zip file (or each JSON file) (red arrow) and hit `Open`.

for windows
```powershell
PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT
```