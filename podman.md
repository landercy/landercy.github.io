podman 在windows中，是在WSL里安装了一个PODMAN的容器。
命令只能在该容器里执行，受限比较明显，没有vim等常用命令。

```bash
podman machine init
podman machine start

/etc/containers/registries.conf