- perform reconnaissance against endpoints/hosts that accept `NTLM` authentication by parsing the information returned within the `CHALLENGE_MESSAGE` (review its other fields to know why this is possible)
- it send `NEGOTIATE_MESSAGE`, then the server respond with `CHALLENGE_MESSAGE`. this challenge message contain infromations that this tool extract them.
```
DumpNTLMInfo.py 172.16.117.3
```