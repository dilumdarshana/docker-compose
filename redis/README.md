# Redis related composes

```bash
# start conainers
$ docker-compose -f replica-docker-compose.yml up
```

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