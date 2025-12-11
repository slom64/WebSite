I had an issue with docker when i do tunnel and the target subnet is the same subnet as docker. The solution is to make docker map different containers to custom subnet

1. Add Custom subnet to `/etc/docker/daemon.json`
```json
{
  "default-address-pools": [
    { "base": "172.31.0.0/16", "size": 24 }
  ]
}

```
2. Restart the container, NOT DOCKER
```sh
docker network prune # remove unused networks
docker compose down -v
docker compose up -d
```


> [!Danger] 
> I have spent alot of time debugging just because i was restarting docker it self instead of restarting the container alone. Restarting the container is more stable to make sure that changes have been done.
