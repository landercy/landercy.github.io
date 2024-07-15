# DOCKER COMPOSE

```bash
# server
mkdir -m 755 -p /data/apps/redis
cd /data/apps/redis

tee docker-compose.yml << EOF
networks:
  work_net:
   external: true
services:
   redis:
     container_name: redis
     image: redis:5.0
     restart: always
     networks:
       work_net:
     ports:
       - '6379:6379'
     volumes:
       - /etc/localtime:/etc/localtime
       - ./data:/data
       - ./redis.conf:/etc/redis.conf
     environment:
       TZ: Asia/Shanghai
     command: redis-server /etc/redis.conf
EOF

tee redis.conf << EOF
port 6379
bind 0.0.0.0
daemonize no
requirepass Admin123

save 60 100
save 300 1
rdbcompression yes
EOF
chmod 644 redis.conf

docker-compose pull
docker-compose up -d

# client for ubuntu
apt install -y redis-tools
```
