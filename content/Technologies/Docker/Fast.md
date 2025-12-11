
| Command                 | Description                                                                                                                            |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `ps -a`                 | To List All Juice Shop Containers (even stopped ones)                                                                                  |
| `stop <container_id>`   | To stop a running container.                                                                                                           |
| `rm <container_id>`     | To remove/delete a docker container(only if it stopped).                                                                               |
| `image ls`              | To see the list of all the available images with their tag, image id, creation time and size.                                          |
| `rmi <image_id>`        | To delete a specific image.<br>`rm -f (docker ps -a \| awk '{print$1}')`: To delete all the docker container available in your machine |
| `image rm <image_name>` | To delete a specific image                                                                                                             |
| `system prune -a`       | To clean the docker environment, removing all the containers and images.                                                               |


Allow network and mount this path to /app in the container
```
sudo docker run -it --rm --network=host -v "$HOME/Documents/exploits/file_upload":/app -w /app docker.image
```