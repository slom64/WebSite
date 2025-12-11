---
tags:
  - SUID
  - pcap
---

## SUID
The main 2 ideas of this machine are:
- even when file has `SUID` that doesn't mean you will get root by just running it. You need to elevate your privileges inside the program by doing `setuid(0)` or `setgid`.
- The `SUID` was given to the file using `capabilities` not directly by assigning it with `SUID`bit.