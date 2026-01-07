
| Command                                                                       | Description                                          |
| ----------------------------------------------------------------------------- | ---------------------------------------------------- |
| `docker volume create <myVol>`                                                | Create Volume                                        |
| `docker container run -it -v <myVolume>:</Container/Path> <Container> <bash>` | Create Container that use `myVolume` as its storage. |
| `docker container run -it -v <Host/Path>:</Docker/Path> <Container> <bash>`   | Mounting                                             |
| `docker container cp </Host/PATH> <Container-ID>:</Container/PATH>`           | Copy file from Host computer to docker container.    |

---
## Docker Storage
- Its hard to write code and scripts inside the container, so it would be better if we can write scripts outside the container then transfer scripts to the container to execute it.
- Container may have persistence / non-persistence data.
### non-persistence data
- When we create new container from image, docker adds additional Read/Write layer, That contain everything we added to the container. So when we remove container all the additional data is removed.
- We can see those additional data on `/var/lib/docker/overlay2/<bigString>/diff`.
- In order to have data that can be changed and never effected by the container will use volume ,bind mount.

> [!NOTE] 
> `overlay2` is different from overlay in networking "overlay", they just have the same name.

---
## Volumes
- Its directroy created in `/var/lib/docker/volumes/`.
- new data will be in `/var/lib/docker/volumes/<MyVol>/_data`, which you can access from host and make changes that container also can see.

---
## Mount
- Mount enable docker to access data from the host.

```
docker container run -it -v /tmp/abc:/app/code python bash
```
---

## Copy
- Not recommanded for developing scripts.
```sh
docker container cp </Host/PATH> <Container-ID>:</Container/PATH>
```