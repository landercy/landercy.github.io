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
```

# COMMAND


```bash
# Database related
show dbs
use <database>
db.dropDatabase()
# 注意  没有插入文档的数据库，在执行`show dbs`时不可见

# Collection related
show collections
db.getCollectionInfos({name:{$regex:’test’}})
db.createCollection(<name>)
db.<collection>.drop()

# Document related
db.<collection>.countDocuments()
db.<collection>.estimatedDocumentCount()

db.<collection>.find()
db.<collection>.findOne()
db.<collection>.find().pretty()
## Conditions
find(fieldName: <value>)
find(fieldName: {<OPERATOR>: <value>})
find(fieldName: {$ne: null})
find(fieldName: {$exists: false})

db.<collection>.find().sort({fieldName:1}).limit(1).skip(1)

db.<collection>.insert({<key>:<value>}) # 主键存在，则报错 E11000 duplicate key error collection
db.<collection>.save({<key>:<value>}) # 主键存在，则全量更新
# 更新文档（全量更新）
# options: {
#   upsert: false， 设置为true时，若无匹配记录，则插入新记录
#   multi: false
# }
db.<collection>.update(<query>, <update>, <options>)
# 删除文档
# options: {
#   justOne true
# }
db.<collection>.remove(<query>, <options>)


# Index related
db.<collection>.getIndexKeys()
db.<collection>.getIndexes()
db.<collection>.totalIndexSize()
db.<collection>.totalIndexSize(1)
# 创建索引
# key: { <field>: <order> }
# order: 1 升序 -1 降序
# options : {
#   background: false
#   unique: false
#   name: <string>
#   expireAfterSeconds: <int> TTL索引 启动一个线程，每60s检查一次，超时的记录直接删除。
# }
db.<collection>.createIndex(<key>,<options>)

# For a single-field index, the sort order (ascending or descending) of the index key does not matter because MongoDB can traverse the index in either direction.
# https://www.mongodb.com/docs/manual/core/indexes/create-index/
db.kccu_events.createIndex({'createdTime': 1})

db.<collection>.dropIndex(<index_name>)
# 删除全部自建索引（不会删除`_id_`）
db.<collection>.dropIndexes()
# 重建索引
db.<collection>.reIndex()

# Explain
db.<collection>.find().explain()

FETCH COLLSCAN IXSCAN

db.stats()
db.stats(1024*1024*1024)

db.serverStatus()

var collNames = db.getCollectionNames();for (var i = 0; i < collNames.length; i++) {   var coll = db.getCollection(collNames[i]);    var stats = coll.stats(1024 * 1024);    print(stats.ns, '\t', stats.storageSize); }

db.currentOp()
db.killOp(opid)

db.auth('root','PASSWORD')
db.getUsers()
use admin
show users
db.changeUserPassword('root','PASSWORD')

use DATABASE
db.default_collection.insertOne({})
db.createUser({user:'admin',pwd:'Admin123',roles:[{role:'readWrite',db:'DATABASE'}],mechanisms:["SCRAM-SHA-1"]})
db.updateUser( 'admin', {roles: [{role:'read', db:'DATABASE'}]})
db.runCommand({ rolesInfo: "read", showPrivileges: 1 })

use admin
db.adminCommand(
   {
      shutdown: 1
   }
)
```

## ROLES

role里的角色可以选：
1、Built-In Roles（内置角色）：
a）数据库用户角色：read、readWrite;
b）数据库管理角色：dbAdmin、dbOwner、userAdmin；
c）集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
d）备份恢复角色：backup、restore；
e）所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
f）超级用户角色：root
–这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner、userAdmin、userAdminAnyDatabase）
h）内部角色：__system

2、具体角色：
a）Read ：允许用户读取指定数据库
b）readWrite：允许用户读写指定数据库
c）backup,retore ：在进行备份、恢复时可以单独指定的角色，在db.createUser()方法中roles里面的db必须写成是admin库，要不然会报错
d）dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
e）userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
f）clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
m）readAnyDatabase ：只在admin数据库中可用，赋予用户所有数据库的读权限
n）readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
o）userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限，
q）dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
k）root：只在admin数据库中可用。超级账号，超级权限



# REPLICA SET

```yml
replication:
   replSetName: "my_cluster"
security:
  keyFile: /data/db/keyfile
```

```bash
# Generate keyfile
# BadValue: security.keyFile is required when authorization is enabled with replica sets
# https://www.mongodb.com/docs/manual/tutorial/deploy-replica-set-with-keyfile-access-control/
# all member use the same keyfile or the following message is displayed when adding new member:
# the following nodes did not respond affirmatively: 10.81.16.6:27017 failed with Authentication failed.
openssl rand -base64 512 > keyfile
chmod 400 keyfile
```

```js
// 初始化时，同时加多个节点且没有仲裁节点时，主节点不一定是哪个
// 如果有数据的库被定为从，而空数据被定为主时，从库会报没有数据可同步
// 应该先初始化主节点，然后添加从节点，避免问题

rs.initiate()

rs.reconfig( {
   _id : "my_cluster",
   members: [
      { _id: 0, host: "192.168.0.11:27017" }
   ]
})

// validate replica set configuration
rs.conf()

// ensure the replica set has a primary
rs.status()

rs.add("192.168.0.12:27017")

// 1. Do not run an arbiter on systems that also host the primary or the secondary members of the replica set.
// 2. Avoid deploying more than one arbiter in a replica set. 
rs.addArb("192.168.0.13:27017")

// is Master
rs.isMaster()['ismaster']

// master downgrade to secondary
rs.stepDown()

// remove member
rs.remove()
```
## STATUS

| 名称       | 状态描述                                                                                                                   |
| ---------- | -------------------------------------------------------------------------------------------------------------------------- |
| STARTUP    | 还不是任何集合的活动成员。所有的成员启动在该状态。在STARTUP状态mongod解析副本集配置文档。                                  |
| PRIMARY    | 处于PRIMARY状态的成员是唯一能接受写操作的成员。                                                                            |
| SECONDARY  | 处于SECONDARY状态的成员复制数据存储。数据可用于读，尽管可能比较旧。                                                        |
| RECOVERING | 可以选举。成员要么实施启动自检测，或完成回滚或重新同步的转换。处于维修模式时：db.adminCommand({"replSetMaintenance":true}) |
| STARTUP2   | 成员加入了集合，正运行初始化同步。                                                                                         |
| UNKNOWN    | 成员的状态，正如从集合的另一个成员中所看到的，未知。                                                                       |
| ARBITER    | 仲裁不复制数据，而仅仅参与选举。                                                                                           |
| DOWN       | 该成员，正如从集合的立即你跟一个成员所见，不可达。(not reachable/healthy)，执行： db.shutdownServer()                      |
| ROLLBACK   | 该成员正在实施回滚。数据不可读。                                                                                           |
| REMOVED    | 成员曾今在副本集但随后被移除。                                                                                             |


## FORCE MEMBER TO BE PRIMARY
You can force a replica set member to become primary by giving it a higher members[n].priority value than any other member in the set.

## PREVENT MEMBER TO BE PRIMARY
If you only need to prevent a member from becoming primary, configure a priority 0 member.

```js
cfg = rs.conf()
cfg.members[1].host
cfg.members[1].priority

cfg.members[1].priority = 0
rs.reconfig(cfg)
rs.config()
```

## HIDE MEMBER
> Hidden members are part of a replica set but cannot become primary and are invisible to client applications. Hidden members may vote in elections.

> The most common use of hidden nodes is to support delayed members. 

### DELAYED MEMBER
> Delayed members contain copies of a replica set's data set. However, a delayed member's data set reflects an earlier, or delayed, state of the set.
> Because delayed members are a "rolling backup" or a running "historical" snapshot of the data set, they may help you recover from various kinds of human error. For example, a delayed member can make it possible to recover from unsuccessful application upgrades and operator errors including dropped databases and collections.

# READ WRITE CONCERN

https://www.mongodb.com/docs/manual/reference/command/getDefaultRWConcern/
https://www.mongodb.com/docs/manual/reference/command/setDefaultRWConcern/

```js
db.adminCommand(
   {
     getDefaultRWConcern: 1
   }
)

db.adminCommand(
  {
    setDefaultRWConcern : 1,
    defaultReadConcern: { 
      "level" : "majority"
    },
    defaultWriteConcern: { 
      "w" : "majority"
    }
  }
)
```

# TROUBLE SHOOTING

## not primary and secondaryOk=false

```js
db.getMongo().setReadPref('secondaryPreferred');
```

# 执行 delete_many() 后，磁盘空间未释放

MongoDB 的 delete_many() 方法用于删除满足特定条件的文档。然而，当您执行删除操作时，MongoDB 不会立即释放磁盘空间。这是因为在 MongoDB 中，数据文件是预先分配的，并且不会因为删除了某些文档而缩小。

```js
// 查看可释放空间大小（默认单位：字节）
db.getCollection('batt_history_udan_229043206410080048ADA000A00000D8').stats().wiredTiger["block-manager"]["file bytes available for reuse"] / 1024 / 1024

// 阻塞压缩
db.runCommand({compact: "batt_history_udan_229043206410080048ADA000A00000D8"})
// 在副本集primary上执行需要加 force 选项
db.runCommand({compact: "somecollection", force: true})

// db.runCommand({ shrink: "yourCollectionName" })
```

compact 具体做了什么？
compact does not block MongoDB CRUD Operations on the database it is compacting.
compact blocks these operations:
- db.collection.drop()
- db.collection.createIndex()
- db.collection.createIndexes()
- db.collection.dropIndex()
- db.collection.dropIndexes()
- collMod

compact 一个集合，会加集合所在DB的互斥写锁，会导致该DB上所有的读写请求都阻塞；因为 compact 执行的时间可能很长，跟集合的数据量相关，所以强烈建议在业务低峰期执行，避免影响业务。
Compact 动作最终由存储引擎 WiredTiger 完成，WiredTiger 在执行 compact 时，会不断将集合文件后面的数据往前面空闲的空间写，然后逐步 truancate 文件回收物理空间。每一轮 compact 前，WT 都会先检查是否符合 comapact 条件。
- 前面80%的空间里，是否有20%的空闲空间，用于写入文件后面20%的数据，或者
- 前面90%的空间里，是否有10%的空闲空间，用于写入文件后面10%的数据

如果上面都不满足，说明执行compact肯定无法回收10%的物理空间，此时 compact 就回退出。
所以有时候遇到对一个大集合进行 compact，compact立马就返回ok，集合的物理空间也没有变化，就是因为 WiredTiger 认为这个集合没有 compact 的必要。


## too many open files

```bash
# 内核可分配的最大文件数
cat  /proc/sys/fs/file-max
1635382
# 单进程可分配的最大文件数
cat /proc/sys/fs/nr_open
1048576
# 用户最大文件句柄
ulimit -n
vim /etc/security/limits.conf
* soft nofile 102400
* hard nofile 102400
reboot

# 进程最大文件句柄
ps -ef|grep -i mongod
cat /proc/<PID>/limits
# 进程当前文件句柄
# ulimit -n: 65535 fd 超过65535时 进程会挂
ls /proc/<PID>/fd | wc -l

# 所有进程文件句柄
cd /proc
for pid in [0-9]*
do
    echo "PID = $pid with $(ls /proc/$pid/fd/ | wc -l) file descriptors"
done