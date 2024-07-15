# DOCKER COMPOSE

```bash
# mongo60
mkdir -m 755 -p /data/apps/mongo
cd /data/apps/mongo

tee mongod.conf << EOF
net:
   bindIp: 127.0.0.1
   port: 27017
storage:
   dbPath: /data/db
systemLog:
   destination: file
   path: /data/logs/mongod.log
   logAppend: false
EOF
chmod 644 mongod.conf

tee docker-compose.yml << EOF
version: '3'
networks:
  work_net:
    external: true
services:
  mongo:
    container_name: mongo
    image: mongo:6.0
    restart: always
    networks:
      work_net:
    ports:
      - '27017:27017'
    volumes:
      - ./data:/data/db
      - ./logs:/data/logs
      - ./mongod.conf:/etc/mongod.conf
    environment:
      TZ: Asia/Shanghai
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: Admin123
    command: -f /etc/mongod.conf
EOF

docker-compose pull
docker-compose up -d

PACKAGE=mongodb-mongosh_1.6.2_amd64.deb
curl -Ok https://10.81.104.11/apps/$PACKAGE
dpkg -i $PACKAGE
rm -f $PACKAGE
```

# 修改root密码

```bash
mongosh -u root

show dbs
use admin

show users
db.changeUserPassword('root','<PASSWORD>')
exit
```

# 创建账号

```bash
use <DATABASE>

# 创建数据库
db.default_collection.insertOne({})

# 业务账号，读写
db.createUser({user: '<USER>', pwd: '<PASSWORD>', roles: [{role: 'readWrite', db: '<DATABASE>'}]})