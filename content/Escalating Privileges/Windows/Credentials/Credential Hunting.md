# LaZenga

 we can run the [LaZagne](https://github.com/AlessandroZ/LaZagne) tool in an attempt to retrieve credentials from a wide variety of software. Such software includes web browsers, chat clients, databases, email, memory dumps, various sysadmin tools, and internal password storage mechanisms (i.e., Autologon, Credman, DPAPI, LSA secrets, etc.).
 
## Running All LaZagne Modules
As we can see, there are many modules available to us. Running the tool with `all` will search for supported applications and return any discovered cleartext credentials.

```powershell
PS C:\htb> .\lazagne.exe all
```
