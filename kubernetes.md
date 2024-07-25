# k8s_node container

```yml
networks:
  work_net:
    external: true
services:
  k8s_master:
    container_name: k8s_master
    image: ubuntu:22.04
    networks:
      work_net:
    command: tail -f /dev/null
  k8s_node1:
    container_name: k8s_node1
    image: ubuntu:22.04
    networks:
      work_net:
    command: tail -f /dev/null
  k8s_node2:
    container_name: k8s_node2
    image: ubuntu:22.04
    networks:
      work_net:
    command: tail -f /dev/null
```


# disable swap

```sh
sudo swapoff -a
```

