# DOCKER COMPOSE

```sh
mkdir -m 755 -p /data/apps/mysql57
cd /data/apps/mysql57

tee my.cnf << EOF
[mysql]
default-character-set  =  utf8mb4

[mysqldump]
quick
quote-names
max_allowed_packet = 64M

[mysqld]
datadir         = /var/lib/mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
log-error       = /var/log/mysql/error.log

skip-host-cache
skip-name-resolve

symbolic-links  = 0
lower_case_table_names = 1
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

slow_query_log  = 1
slow_query_log_file   =  /var/log/mysql/slow.log
long_query_time = 1

character-set-filesystem = binary
character-set-server     = utf8mb4
EOF
chmod 644 my.cnf

tee docker-compose.yml << EOF
networks:
  work_net:
    external: true
services:
  mysql57:
    container_name: mysql57
    image: mysql:5.7
    restart: always
    networks:
      work_net:
    ports:
      - '3306:3306'
    volumes:
      - /etc/localtime:/etc/localtime
      - /var/run/mysqld:/var/run/mysqld
      - ./data:/var/lib/mysql
      - ./my.cnf:/etc/my.cnf
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: 'Admin123'
EOF

docker-compose pull
docker-compose up -d

apt install -y mysql-client-core-8.0 
```

## 修改ROOT密码

```sh
docker exec -it mysql57 mysql -uroot -p
```

```sql
-- for MYSQL 5.7
SET PASSWORD FOR root@'%' = PASSWORD('<PASSWORD>');
SET PASSWORD FOR root@localhost = PASSWORD('<PASSWORD>');

-- for MySQL8
ALTER USER root@'%' IDENTIFIED BY '<PASSWORD>'; 
ALTER USER root@localhost IDENTIFIED BY '<PASSWORD>'; 
```

## 创建数据库和账号

```sql
-- 创建数据库
CREATE DATABASE <DATABASE> DEFAULT CHARSET utf8mb4;

-- 业务账号，读写
CREATE USER <USER>@'%' IDENTIFIED by '<PASSWORD>';
GRANT SELECT, INSERT, UPDATE, DELETE ON <DATABASE>.* TO <USER>@'%';

-- 业务账号，只读
CREATE USER <USER>_ro@'%' IDENTIFIED by '<PASSWORD>';
GRANT SELECT ON <DATABASE>.* TO <USER>_ro@'%';

FLUSH PRIVILEGES;
```

# 数据备份和还原

```bash
# export
mysqldump -h <host> -u <user> -p --default-character-set=utf8mb4 <database> | gzip > <filename>.sql.gz
# import
gunzip < <filename>.sql.gz | mysql -h <host> -u <user> -p --default-character-set=utf8mb4 <database>
```