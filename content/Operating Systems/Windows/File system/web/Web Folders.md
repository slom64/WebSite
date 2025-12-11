When you’re on a **Windows machine that’s running a website** (IIS, Apache, etc.), the actual website files (HTML, ASPX, PHP, etc.) are stored on disk in specific directories.

It depends on the **web server software**:

---

## 🔹 1. IIS (Internet Information Services — Microsoft’s built-in web server)

Default root folders:

- **Website files:**
    ```
    C:\inetpub\wwwroot\
    ```
    
    That’s the default root for the “Default Web Site”.  
    If multiple sites exist, they can be in subfolders like:
    
    ```
    C:\inetpub\wwwroot\site1\
    C:\inetpub\wwwroot\site2\
    ```
    
- **IIS configuration:**
    
    ```
    C:\Windows\System32\inetsrv\config\
    ```
    
    (main file is `applicationHost.config`)
    
- **IIS logs:**
    
    ```
    C:\inetpub\logs\LogFiles\
    ```
    

---

## 🔹 2. Apache on Windows

If Apache is installed, the config decides the root, but defaults are usually:

- **Website files:**
    
    ```
    C:\Program Files\Apache Group\Apache2\htdocs\
    ```
    
    or
    
    ```
    C:\xampp\htdocs\
    ```
    
- **Apache config:**
    
    ```
    C:\Program Files\Apache Group\Apache2\conf\httpd.conf
    ```
    

---

## 🔹 3. XAMPP / WAMP (common in labs)

These packages bundle Apache, PHP, and MySQL.

- **Website files (XAMPP):**
    
    ```
    C:\xampp\htdocs\
    ```
    
- **Website files (WAMP):**
    
    ```
    C:\wamp\www\
    ```
    

---

## 🔹 4. Custom setups

Sometimes admins point the website root to another drive (e.g. `D:\websites\corpportal\`).  
To find out exactly where:

- Check IIS bindings in `inetmgr` (GUI) or `applicationHost.config`.
    
- For Apache, look in `httpd.conf` → the `DocumentRoot` directive.
    

---

✅ **So “where do website things live in Windows?”**

- IIS: `C:\inetpub\wwwroot\`
    
- XAMPP: `C:\xampp\htdocs\`
    
- WAMP: `C:\wamp\www\`
    
- Apache: `...\htdocs\` under install path
    
- Custom paths possible (check config files).
    

---

👉 Do you want me to also show you how to **enumerate these paths remotely** (like via webshell, SMB, or command injection) in an AD/HTB lab, so you don’t have to guess?