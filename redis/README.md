# Redis related composes

```bash
# start conainers
$ docker-compose -f replica-docker-compose.yml up
```

## Redis Cluster

6 nodes (3 masters + 3 replicas) on a dedicated `172.22.0.0/24` network.

```bash
# start all nodes
$ docker compose -f cluster-docker-compose.yml up -d

# create the cluster once (after all nodes are up)
$ docker exec -it redis-cluster-1 redis-cli --cluster create \
    redis-cluster-1:6379 redis-cluster-2:6379 redis-cluster-3:6379 \
    redis-cluster-4:6379 redis-cluster-5:6379 redis-cluster-6:6379 \
    --cluster-replicas 1

# connect with cluster mode
$ redis-cli -c -p 7001
```

Host ports 7001-7006 map to each node's internal 6379. Data persists in `data/cluster-<n>/`.

## Test sentinels works

```bash

# make sure everything works
$ docker exec sentinel-1 redis-cli -p 26379 SENTINEL sentinels mymaster
# this should show other 2 sentinels in the output. If shows all of sentinels in the list, which means all sentinels knows each other

# stop and remove all containers
$ docker-compose -f replica-docker-compose.yml down

# check who is the master
$ docker exec sentinel-1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
# this should return our orignal master node ip address and port

# Test master down scenario

# force to down master
$ docker stop redis-master

# after few seconds, one of the replica should promoted as new master
$ docker exec sentinel-1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster

# now, start the stopped old master
$ docker start redis-master

# check role of the master
$ docker exec redis-master redis-cli INFO replication
# role: slave

# yey... all works
```