# Redis Sentinel Docker Compose

A production-like Redis high availability setup using **Redis master-slave replication with Sentinel**, all managed via Docker Compose.

## Environment

- **OS**: Ubuntu 22.04.3 LTS
- **Docker Compose**: v2.26.1
- **Redis**: 7.2.4 (master-slave replication)
- **Redis Sentinel**: 3 nodes for automatic failover


## Directory Structure
### Before Starting
```shell
docker-redis
├── data
├── redis
│   ├── redis-1.conf
│   ├── redis-2.conf
│   └── redis-3.conf
└── sentinel
    ├── sentinel-1
    │   └── sentinel-1.conf
    ├── sentinel-2
    │   └── sentinel-2.conf
    └── sentinel-3
        └── sentinel-3.conf
```

### After Starting
```shell
docker-redis
├── data
│   ├── redis-1_dump.rdb
│   ├── redis-1.log
│   ├── redis-1.pid
│   ├── redis-2_dump.rdb
│   ├── redis-2.log
│   ├── redis-2.pid
│   ├── redis-3_dump.rdb
│   ├── redis-3.log
│   ├── redis-3.pid
│   ├── sentinel-1.log
│   ├── sentinel-2.log
│   └── sentinel-3.log
├── redis
│   ├── redis-1.conf
│   ├── redis-2.conf
│   └── redis-3.conf
└── sentinel
    ├── sentinel-1
    │   └── sentinel-1.conf
    ├── sentinel-2
    │   └── sentinel-2.conf
    └── sentinel-3
        └── sentinel-3.conf
```


## Usage Steps

### 1. Create project directories
```shell
mkdir -p docker-redis/{data,redis,sentinel}
mkdir -p docker-redis/sentinel/{sentinel-1,sentinel-2,sentinel-3}
```

### 2. Create a dedicated Docker network
```shell
docker network create --driver bridge airflow_cluster
```

### 3. Edit docker-compose.yaml
- Adjust the port mappings as needed
- Update timezone in the environment section if required

### 4. Prepare configuration files
You can either:
- Use the example configuration files provided in this repository, or
- Download Redis configuration files from the [Redis official website](https://raw.githubusercontent.com/redis/redis/7.2/redis.conf)

### 5. If using example config files in this repository, skip to step 8.
Otherwise, if you downloaded from Redis, copy the config file 3 times:
```shell
cp redis.conf redis/redis-1.conf
cp redis.conf redis/redis-2.conf
cp redis.conf redis/redis-3.conf
```

### 6. Edit each Redis config file
Modify the following lines in each file:
```conf
# bind 127.0.0.1 ::1   ← Comment this out

protected-mode no

pidfile ./redis-1.pid         # Adjust the number per instance

logfile ./redis-1.log         # Adjust the number per instance

dbfilename redis-1_dump.rdb   # Adjust the number per instance

dir ./
```

Repeat for redis-2.conf and redis-3.conf with corresponding filenames.

### 7. Edit Sentinel config files
Create or edit the following files:
```shell
vi sentinel/sentinel-1/sentinel-1.conf
vi sentinel/sentinel-2/sentinel-2.conf
vi sentinel/sentinel-3/sentinel-3.conf
```

Each file should contain:
```conf
port 26379

sentinel monitor mymaster <your_host_IP> 6380 2

sentinel down-after-milliseconds mymaster 10000

sentinel failover-timeout mymaster 60000

sentinel parallel-syncs mymaster 1

protected-mode no

logfile /data/sentinel-1.log   # Change according to instance

dir ./

SENTINEL resolve-hostnames no

SENTINEL announce-hostnames no
```

### 8. Start the cluster
From the docker-redis directory, run:
```shell
docker compose up -d
```