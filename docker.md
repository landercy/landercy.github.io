# DAEMON.JSON

```json
{
    "data-root": "/data/apps/docker",
    "registry-mirrors": [
        "https://dockerhub.icu",
        "https://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ],
    "insecure-registries": ["registry.local"]
}
```

# RUN DOCKER COMMAND WITHOUT SUDO
```sh
sudo groupadd docker
sudo gpasswd -a ${USER} docker
logout
```

# 修改docker目录
vim <path_to_docker>/containers/<hash_code>/config.v2.json

:s/\/var\/lib\/docker/\/data\/apps\/docker/g

docker run -it --rm --name=<name> -p <port>:<port> -v <path>:<path> <image>:<version>


# REGISTRY

## TLS证书

```sh
# 创建自签证书
cd /root/apps/docker-registry
mkdir -p certs
# CN(Common Name, eg, hostname) 必填
openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt

# 自签证书白名单
mkdir -p /etc/docker/certs.d/HOST
cp certs/domain.crt /etc/docker/certs.d/HOST/ca.crt
# no restart needed
```

## DOCKER-COMPOSE

```yml
version: '3'
networks:
  work_net:
    external: true
services:
  docker-registry:
    container_name: docker-registry
    image: registry:2.7
    # restart: always
    networks:
      work_net:
    ports:
      - '443:5000'
    environment
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    volumes:
      - ./data:/var/lib/registry
      - ./certs:/certs
```

```sh
docker-compose up -d
docker pull alpine:3.12
docker image tag alpine:3.12 n11.vm/alpine
docker push n11.vm/alpine
docker pull n11.vm/alpine
```

# DOCKERFILE

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y openjdk-11-jdk

WORKDIR /opt

COPY ./opt/ /opt/

EXPOSE 8000

CMD ["bash", "run.sh"]
```

```yml
networks:
  work_net:
    external: true
services:
  app_jdk11:
    container_name: app_jdk11
    image: app_jdk11
    # restart: unless-stopped
    networks:
      work_net:
    ports:
      - 8000:8000
    volumes:
      - /etc/localtime:/etc/localtime
      - ./opt:/opt
    environment:
      TZ: Asia/Shanghai
```

# COMMAND

# 指定容器 IP

```bash
docker network create --subnet=172.24.0.0/16 spark_network
```

## overlay占用大量空间

```bash
# 查询磁盘占用情况
docker system df 
# 清理空间，慎用
docker system prune -a

cd /data/apps/docker/overlay2
du -BG --max-depth=1 | sort -nr | head -6

docker ps -q | xargs docker inspect --format '{<!-- -->{.State.Pid}}, {<!-- -->{.Id}}, {<!-- -->{.Name}}, {<!-- -->{.GraphDriver.Data.WorkDir}}' | grep  '<OVERLAY>'
```