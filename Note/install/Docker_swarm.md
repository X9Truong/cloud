### Docker swarm

`docker node ls`

`docker service ls`

`docker service ps`

`docker service inspect ID`

`docker service logs ID`

`docker service rm ID`

`docker service scale up`

`docker service create nginx`

`docker service update ID --replicas 10` 

### Create docker swarm

`docker swarm init --advertise-addr 192.168.99.121`

`docker service create --replicas 10 nginx`

`docker service ps ID`



