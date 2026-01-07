
| Command                        | Description                                                                                                                            |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `ps -a`                        | To List All Juice Shop Containers (even stopped ones)                                                                                  |
| `stop <container_id>`          | To stop a running container.                                                                                                           |
| `rm <container_id>`            | To remove/delete a docker container(only if it stopped).                                                                               |
| `image ls`                     | To see the list of all the available images with their tag, image id, creation time and size.                                          |
| `rmi <image_id>`               | To delete a specific image.<br>`rm -f (docker ps -a \| awk '{print$1}')`: To delete all the docker container available in your machine |
| `image rm <image_name>`        | To delete a specific image                                                                                                             |
| `system prune -a`              | To clean the docker environment, removing all the containers and images.                                                               |
| `-d` X `-it`                   | -d: detached mode, -it: `i` for interactive, `t` for terminal                                                                          |
| `-e`                           | Pass env variables                                                                                                                     |
| `-p <Host-Port>:<Docker-Port>` | Port mapping -p `<Host-Port>:<Docker-Port>`                                                                                            |
| ctrl + p q                     | exits without killing the container.                                                                                                   |


Allow network and mount this path to /app in the container
```
sudo docker run -it --rm --network=host -v "$HOME/Documents/exploits/file_upload":/app -w /app docker.image
```

Course commands
```sh
docker image pull fedora
docker image ls
docker container create -it fedora bash
docker container ls -a
docker container start -i HASH_fedor

# This is same as `docker container create` +  `docker container start` and in addition if the image isn't existed in local, it will automatically pull it.
# run = pull + create + start  
docker container run 
```

Notes
- Docker start with only one process, so when you exit from terminal of interactive terminal, you are killing the main process so the whole container finished. So container = command/application and if you terminate the command/application you are terminating the container. Example, When you start fedora container, you have added some specification obove your kernel and you have start using this fedora using bash. "bash" here acts as command/application when it dies the container dies too. the `PID=1` is the main process that we have started our container with and if it dies the container dies to. That doesn't mean we can't run other process, we can have `PID=2,3,4` but those process doesn't effect the state of the container. Only `PID=1` do. 
```sh
# start python3 container but instead of intercating using python3 shell, we start with bash shell
docker container run -it python3 /bin/bash
```

- Docker implements layer concept when pulling images, why? because if there is a layer is used in different images then docker will reuse the same layer in different images "There is hash for each layer so docker use it to know if the layer is used else where", So it is best practice to not squash "squeeze" the image to single layer because that reduce reusability which make us use more space and network traffic. 