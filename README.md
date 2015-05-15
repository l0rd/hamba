# Docker Ambassador Pattern

Docker container usefull to setup a simple, effective ambassador

## Usage Example

```
 /-------------------------\      /-------------------------\      /-------------------------\                                                
 |  Provider               |      | Ambassador              |      | Consumer                |                                
 |  (redis)                |      | (flask_amba)            |      | (flask)                 |                                
 |  Exposes port: 6379     |<---->| Exposes port: 6379      |<---->| Exposes port: 5000:5000 |                                               
 |  IP address may change  |      | IP address don't change |      | Connect to port 6379 of |                                
 |and can be on remote host|      |                         |      |  redis host             |                                
 \-------------------------/      \-------------------------/      \-------------------------/                                
```

### Initial Setup
```bash
# Start the provider (redis)
docker run -d --name redis -p 6379 redis
export REDIS_IP=$(docker inspect --format '{{ json .NetworkSettings.IPAddress }}' redis | sed -e 's/^"//'  -e 's/"$//')
echo $REDIS_IP

# Start the ambassador (flask_amba)
export FRONT_PORT=6379
export BACK_PORT=6379
export BACK_IP=$REDIS_IP
docker run --name flask_amba -p 6379 -d jpetazzo/hamba $FRONT_PORT $BACK_PORT $BACK_IP

# Start the consumer (flask)
docker run -d --name flask -p 5000:5000 --link flask_amba:redis jpetazzo/figdemo
```

### When the provider IP address changes
```bash
export FRONT_PORT=6379
export BACK_PORT=6379
export BACK_IP=$REDIS_IP
docker exec -d flask_amba hamba $FRONT_PORT $BACK_PORT $BACK_IP
```
