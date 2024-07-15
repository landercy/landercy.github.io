# DOCKER COMPOSE

```yml
networks:
  work_net:
    external: true
services:
  rabbitmq:
    container_name: rabbitmq
    image: rabbitmq:3.10-management-alpine
    restart: always
    networks:
      work_net:
    ports:
      - '5672:5672'
      - '15672:15672'
    environment:
      RABBITMQ_DEFAULT_USER: 'admin'
      RABBITMQ_DEFAULT_PASS: 'Admin123'
```

# COMMAND

```bash
# new user
rabbitmqctl add_user <user> <password>
# list users
rabbitmqctl list_users
# assign roles
rabbitmqctl set_user_tags <user> <tag>
# add host
rabbitmqctl add_vhost <host_path>
# grant permissions
rabbitmqctl set_permissions -p <vhost_path> <user> <config preg> <write preg> <read preg>
# list permissions
rabbitmqctl list_permissions [-p <vhost_path>]
# list user permissions
rabbitmqctl list_user_permissions <user>
# clear permission
rabbitmqctl clear_permissions [-p <vhost_path>] <user>

delete_user <user>
change_password <user> <new_password>
clear_password <user>

rabbitmqctl add_vhost vhostpath
rabbitmqctl list_vhosts
rabbitmqctl list_bindings
rabbitmqctl list_exchanges
rabbitmqctl list_queues

rabbitmqadmin -u {user} -p {password} -V {vhost} declare exchange name={name} type={type}
rabbitmqadmin -u {user} -p {password} -V {vhost} declare queue name={name}
rabbitmqadmin -u {user} -p {password} -V {vhost} declare binding source={Exchange} destination={queue} routing_key='{key}'

rabbitmq-plugins directories -s
Plugin archives directory: /opt/rabbitmq/plugins

download plugins.ez into /opt/rabbitmq/plugins
rabbitmq-plugins list
rabbitmq-plugins enable plugin_name
```

# CONCEPTS

## PORTS

By default, RabbitMQ will listen on port 5672 on all available interfaces.
4369: epmd, a peer discovery service used by RabbitMQ nodes and CLI tools
5672, 5671: used by AMQP 0-9-1 and AMQP 1.0 clients without and with TLS
5552, 5551: used by the RabbitMQ Stream protocol clients without and with TLS
6000 through 6500: used by RabbitMQ Stream replication
25672: used for inter-node and CLI tools communication (Erlang distribution server port) and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000). Unless external connections on these ports are really necessary (e.g. the cluster uses federation or CLI tools are used on machines outside the subnet), these ports should not be publicly exposed. See networking guide for details.
35672-35682: used by CLI tools (Erlang distribution client ports) for communication with nodes and is allocated from a dynamic range (computed as server distribution port + 10000 through server distribution port + 10010). See networking guide for details.
15672, 15671: HTTP API clients, management UI and rabbitmqadmin, without and with TLS (only if the management plugin is enabled)
61613, 61614: STOMP clients without and with TLS (only if the STOMP plugin is enabled)
1883, 8883: MQTT clients without and with TLS, if the MQTT plugin is enabled
15674: STOMP-over-WebSockets clients (only if the Web STOMP plugin is enabled)
15675: MQTT-over-WebSockets clients (only if the Web MQTT plugin is enabled)
15692: Prometheus metrics (only if the Prometheus plugin is enabled)

# USER TAG
1.management：用户可以访问管理插件
2.policymaker：用户可以访问管理插件并管理他们有权访问的虚拟主机的策略和参数
3.monitoring：用户可以访问管理插件并查看所有连接和通道以及节点相关信息。
4.administrator：用户可以做任何监控可以做的事情，管理用户、虚拟主机和权限，关闭其他用户的连接，以及管理所有虚拟主机的策略和参数。