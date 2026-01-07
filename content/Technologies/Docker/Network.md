- Docker by default has `bridge`, `host`, `none` networks. 
	- `bridge/ NAT "windows"`: by default containers created in bridge network. So containers can talk to each other.
	- `host`: Docker behave as the host, and docker now have all the host adapter, docker now is exposed as if it was the host.
	- `None`: container isolated from everything even from the host and it doesn't have even adapters.
- for each container it will be created virtual adapter on host.
- Docker has drivers for each network.
	- `bridge`: This is used in bridge network.
	- `host`: used in host network, You only have one network that use host driver.
	- `null`: used in none network.
	- `macvlan`: layer 2 network driver, makes the docker behave is it was physical device in the network.
	- `overlay`: Makes containers talk to each other on different hosts, used in docker swarm.


| Command                                                                                                                            | Description                                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `--network <NetworkName/bridge/mynet>`<br>                                                                                         | use none or host driver network                                                               |
| `docker network create <NetworkName/mynet> -d <driver/bridge> --subnet <10.10.10.10/16>`<br>`docker network create ... --internal` | create new network.<br>containers can't reach internet, but reach each other.<br>             |
| `docker network connect <Network/Name> <Container>`<br>`docker network disconnect <Network/Name> <Container>`                      | Connect running containers to specific network<br>Disconnect container from specific network. |
|                                                                                                                                    |                                                                                               |
