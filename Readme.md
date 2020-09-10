
### Introduction
   I have configured redis cluster with HA by using sentinals and i have used clinet which is writen on GO

### Configuration

```
#persistence
dir /data
dbfilename dump.rdb
appendonly yes
appendfilename "appendonly.aof"

```
### redis-0 Configuration

```
protected-mode no
port 6379

#authentication
masterauth Admin
requirepass Admin
```
### redis-1 Configuration

```
protected-mode no
port 6379
slaveof redis-0 6379

#authentication
masterauth Admin
requirepass Admin

```
### redis-2 Configuration

```
protected-mode no
port 6379
slaveof redis-0 6379

#authentication
masterauth Admin
requirepass Admin

```

```

### Installation ( using Docker)

docker network create redis

cd clustering\

#redis-0
docker run -d --rm --name redis-0 `
    --net redis `
    -v ${PWD}/redis-0:/etc/redis/ `
    redis:6.0-alpine redis-server /etc/redis/redis.conf

#redis-1
docker run -d --rm --name redis-1 `
    --net redis `
    -v ${PWD}/redis-1:/etc/redis/ `
    redis:6.0-alpine redis-server /etc/redis/redis.conf


#redis-2
docker run -d --rm --name redis-2 `
    --net redis `
    -v ${PWD}/redis-2:/etc/redis/ `
    redis:6.0-alpine redis-server /etc/redis/redis.conf

```

##  Application

```
cd client\
docker build . -t aimvector/redis-client:v1.0.0

docker run -it --net redis -e REDIS_HOST=redis-0 -e REDIS_PORT=6379 -e REDIS_PASSWORD="a-very-complex-password-here" -p 80:80 shivprasad/redis-client:v1.0.0

```

## Test Replication

Technically written data should now be on the replicas

```
# go to one of the clients
docker exec -it redis-2 sh
redis-cli
auth "Admin"
keys *

```

## Instlattion of Sentinels

```
#BASIC CONFIG
port 5000
sentinel monitor mymaster redis-0 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
sentinel auth-pass mymaster Admin
#********************************************

```
Starting Redis in sentinel mode

```
cd clustering\

docker run -d --rm --name sentinel-0 --net redis `
    -v ${PWD}/sentinel-0:/etc/redis/ `
    redis:6.0-alpine `
    redis-sentinel /etc/redis/sentinel.conf

docker run -d --rm --name sentinel-1 --net redis `
    -v ${PWD}/sentinel-1:/etc/redis/ `
    redis:6.0-alpine `
    redis-sentinel /etc/redis/sentinel.conf

docker run -d --rm --name sentinel-2 --net redis `
    -v ${PWD}/sentinel-2:/etc/redis/ `
    redis:6.0-alpine `
    redis-sentinel /etc/redis/sentinel.conf


docker logs sentinel-0
docker exec -it sentinel-0 sh
redis-cli -p 5000
info
sentinel master mymaster
