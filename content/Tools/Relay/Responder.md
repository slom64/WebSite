Altrantives: https://github.com/RedTeamPentesting/pretender pretender
- There are other options in the Responder configuration file, like `Specific IP Addresses to respond to (default = All)`, which can be helpful to configure in some cases where we only want to relay connections from a specific IP or group of IPs.
-  the file of configurations `Responder.conf`.
- You can't spoof/poison `LLMNR`/`NETBIOS`, read this for more information.



| flag | Usage                                                             |
| ---- | ----------------------------------------------------------------- |
| -A   | Analysis mode<br>Recon mode, where we only listen to the traffic. |

```sh
sudo virsh net-destroy default # to enable DNS on your local, it kills virtual network for KVM/QEMU.
sudo python3 Responder.py -I ens192
```


