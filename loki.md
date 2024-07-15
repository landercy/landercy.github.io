# loki

wget https://raw.githubusercontent.com/grafana/loki/v2.9.1/production/docker-compose.yaml -O docker-compose.yaml
docker-compose -f docker-compose.yaml up

# grafana loki

```yml
version: "3"
networks:
  work_net:
    external: true
services:
  grafana_loki:
    container_name: grafana_loki
    image: grafana/loki:2.9.0
    networks:
      - work_net
    ports:
      - "3100:3100"
    volumes:
      - ./data:/loki
      - ./config.yml:/etc/loki/config.yml
    command: -config.file=/etc/loki/config.yml
```

```yml
server:
  http_listen_port: 3100
  http_path_prefix: /loki
auth_enabled: false
analytics:
  reporting_enabled: false
common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2023-10-18
      store: boltdb-shipper
      object_store: filesystem
      schema: v12
      index:
        prefix: index_
        period: 24h
#ruler:
#  alertmanager_url: http://localhost:9093
```

cat my.log | promtail --stdin --dry-run --client.url http://127.0.0.1:3100/loki/api/v1/push

# grafana promtail

```yml
version: "3"
networks:
  work_net:
    external: true
services:
  grafana_promtail:
    container_name: grafana_promtail
    # image: grafana/promtail:2.9.0
    image: registry.local/promtail290
    networks:
      - work_net
    volumes:
      - ./data:/promtail
      - ./config.yml:/etc/promtail/config.yml
      - /data/apps/nginx/logs:/promtail/nginx
    command: -config.file=/etc/promtail/config.yml
```

## config

```yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

# The positions block is needed, it allow Promtail continues from where it left off after a restart.
positions:
  filename: /promtail/positions.yml

clients:
  - url: http://grafana_loki:3100/loki/api/v1/push
    tls_config: 
       insecure_skip_verify: true

scrape_configs:
  - job_name: nginx
    static_configs:
    - labels:
        job: nginx
        # to identify logs from this machine vs others
        host: 10.81.104.11
        __path__: /promtail/nginx/*.log
```

```bash
cd data
ln -s /data/apps/nginx/logs nginx
```