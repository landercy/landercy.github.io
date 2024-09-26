# PODMAN IN WINDOWS

> 不好用

podman 在windows中，是在WSL里安装了一个PODMAN的容器。
命令只能在该容器里执行，受限比较明显，没有vim等常用命令。

```bash
podman machine init
podman machine start

/etc/containers/registries.conf
```


# PODMAIN IN UBUNTU

```sh
sudo apt update -y
sudo apt install podman

# run a container
podman run --name basic_httpd -dt -p 8080:80/tcp docker.io/nginx

# list running containers
podman ps

# check metadata and details of the container
podman inspect basic_httpd | grep IPAddress\":
# As the container is running in rootless mode, an IP address is not assigned and the value will be listed as "none" in the output from inspect.
>  "IPAddress": "",

# test
curl -I http://localhost:8080

# view logs
podman logs basic_httpd

# view pid
podman top basic_httpd

# create a checkpoint
# containers must run with root, or no no such container found
# Error: checkpointing a container requires root
sudo podman container checkpoint basic_httpd
sudo podman container restore  basic_httpd

# stop a container
podman stop basic_httpd

# list all containers
podman ps -a

# remove a container
podman rm basic_httpd
```

# 国内源

```sh
cd /etc/containers/
sudo cp registries.conf registries.conf.bak

sudo tee registries.conf << EOF
unqualified-search-registries = ["docker.io"]

[[registry]]
[[registry.mirror]]
location = "hub-mirror.c.163.com"
[[registry.mirror]]
location = "docker.mirrors.ustc.edu.cn"
[[registry.mirror]]
location = "registry.docker-cn.com"
EOF


```

# EXCEPTIONS

## WARN[0000] "/" is not a shared mount, this could cause issues or missing mounts with rootless containers

```sh
findmnt -o PROPAGATION /
> PROPAGATION
> private

# temporarily
sudo mount --make-shared /

# permanently
sudo vim /etc/fstab
```

| device                                   | dir | type | options         | dump | fsck |
| ---------------------------------------- | --- | ---- | --------------- | ---- | ---- |
| UID=a19f2d38-05b2-4097-9bf7-1007ae780a6d | /   | ext4 | defaults,shared | 0    | 1    |


## Error: error initializing ID mappings: UID setting is malformed expected ["uint32:uint32:uint32"]: ["+g102000:@2000"]