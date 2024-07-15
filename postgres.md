# DOCKER COMPOSE

```bash
mkdir -m 755 -p /data/apps/postgres
cd /data/apps/postgres

tee docker-compose.yml << EOF
version: '3'
networks:
  work_net:
    external: true
services:
  postgres: 
    container_name: postgres
    image: postgres:11
    restart: always
    networks:
      work_net:
    ports:
      - '5432:5432'
    volumes:
      - /etc/localtime:/etc/localtime
      - ./data:/var/lib/postgresql/data
    environment: 
      TZ: Asia/Shanghai
      POSTGRES_USER: 'root'
      POSTGRES_PASSWORD: 'Admin123'
EOF

docker-compose pull
docker-compose up -d

apt install -y postgresql-client
```

## 修改ROOT密码

```sql
ALTER USER ROOT WITH PASSWORD '<PASSWORD>';
```

## 创建统计账号

```sql
CREATE USER stats WITH PASSWORD '<PASSWORD>';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO stats;
```

## 创建数据库和账号

```sql
CREATE DATABASE <DATABASE>;
\c <DATABASE>
CREATE USER <USER> WITH PASSWORD '<PASSWORD>';
GRANT USAGE ON SCHEMA <SCHEMA> TO <USER>;
GRANT SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA <SCHEMA> TO <USER>;
GRANT SELECT, UPDATE, INSERT, DELETE ON ALL TABLES IN SCHEMA <SCHEMA> TO <USER>;
```

# 异常处理

### pg_wal目录体积过大

```bash
dc exec postgres bash
wal_file=$(/usr/local/bin/pg_controldata /var/lib/postgresql/data | grep 'REDO WAL file' | awk '{print $6}') && /usr/local/bin/pg_archivecleanup -d /var/lib/postgresql/data/pg_wal $wal_file
```