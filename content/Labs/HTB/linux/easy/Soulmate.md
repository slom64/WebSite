## Summary
- Recon → 22, 80
- Vhost: ftp.soulmate.htb --> use crushFTP
- CVE-2025-31161 → create admin user
- Inside crush FTP upload reverse shell then triger it → RCE as www-data
- Local Erlang SSH (2222) → CVE-2025-3243
- Skip user privesc → direct **root shell**