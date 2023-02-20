# Redis sentinel 설정

redis master 1 , replica 2, sentinel 3 구성

master가 죽었을경우 자동으로 fail over 를 해준다.

## docker-compose.yml

```yaml
version: '3.8'

services:
  redis-master:
    hostname: redis-master
    container_name: redis-master
    image: bitnami/redis:6.2.6
    environment:
      - REDIS_REPLICATION_MODE=master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6379:6379

  redis-slave-1:
    hostname: redis-slave-1
    container_name: redis-slave-1
    image: bitnami/redis:6.2.6
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6480:6379
    depends_on:
      - redis-master

  redis-slave-2:
    hostname: redis-slave-2
    container_name: redis-slave-2
    image: bitnami/redis:6.2.6
    environment:
      - REDIS_REPLICATION_MODE=slave
      - REDIS_MASTER_HOST=redis-master
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - 6481:6379
    depends_on:
      - redis-master
      - redis-slave-1

  redis-sentinel-1:
    hostname: redis-sentinel-1
    container_name: redis-sentinel-1
    image: bitnami/redis-sentinel:6.2.6
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=master-name
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    ports:
      - 26379:26379

  redis-sentinel-2:
    hostname: redis-sentinel-2
    container_name: redis-sentinel-2
    image: bitnami/redis-sentinel:6.2.6
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=master-name
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    ports:
      - 26380:26379

  redis-sentinel-3:
    hostname: redis-sentinel-3
    container_name: redis-sentinel-3
    image: bitnami/redis-sentinel:6.2.6
    environment:
      - REDIS_SENTINEL_DOWN_AFTER_MILLISECONDS=3000
      - REDIS_MASTER_HOST=redis-master
      - REDIS_MASTER_PORT_NUMBER=6379
      - REDIS_MASTER_SET=master-name
      - REDIS_SENTINEL_QUORUM=2
    depends_on:
      - redis-master
      - redis-slave-1
      - redis-slave-2
    ports:
      - 26381:26379
a
```
