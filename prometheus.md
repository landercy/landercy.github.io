# DOCKER-COMPOSE.YML

```yml
networks:
  work_net:
    external: true
services:
  prometheus:
    container_name: prometheus
    image: prom/prometheus:latest
    restart: unless-stopped
    networks:
      - work_net
    ports:
      - '9090:9090'
    volumes:
      - ./data:/prometheus/data
      - ./config:/prometheus/config
    command: --config.file=/prometheus/config/prometheus.yml
  prometheus_alert_manager:
    container_name: prometheus_alert_manager
    image: prom/alertmanager:latest
    restart: unless-stopped
    networks:
      - work_net
    ports:
      - '9093:9093'
    volumes:
      - ./config:/alertmanager/config
    command: --config.file=/alertmanager/config/alertmanager.yml
```

## PROMETHEUS.YML

```yml
global:
  scrape_interval: 1m
  scrape_timeout: 10s
  evaluation_interval: 1m
rule_files:
  - 'rules/*.yml'
scrape_config_files:
  - 'scrapes/*.yml'
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - prometheus_alert_manager:9093

scrape_configs:
- job_name: GENERIC INFO
  honor_timestamps: true
  scrape_interval: 1m
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: https
  tls_config:
    insecure_skip_verify: true
  follow_redirects: true
  enable_http2: true
  static_configs:
  - targets:
    - 192.168.1.11
```

```bash
# check config
promtool check config prometheus.yml 
```


## RULES/GENERIC_INFO.YML

```yml
groups:
  - name: NODE EXPORTER
    rules:
      - alert: 节点采集器不可用[存在服务器宕机可能性]
        expr: up == 0
        for: 5m
        labels:
          severity: critical
  - name: MEMORY
    rules:
      - alert: 内存可用空间不足10%
        expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 10
        for: 5m
        labels:
          severity: critical
  - name: DISK
    rules:
      - alert: 系统盘可用空间不足25%
        expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100 < 25
        for: 5m
        labels:
          severity: critical
      - alert: 数据盘可用空间不足25%
        expr: node_filesystem_avail_bytes{mountpoint="/data"} / node_filesystem_size_bytes{mountpoint="/data"} * 100 < 25
        for: 5m
        labels:
          severity: waring
      - alert: 数据盘可用空间不足10%
        expr: node_filesystem_avail_bytes{mountpoint="/data"} / node_filesystem_size_bytes{mountpoint="/data"} * 100 < 10
        for: 5m
        labels:
          severity: critical
```

## SCRAPES/GENERIC_INFO.YML

```yml
scrape_configs:
- job_name: GENERIC INFO
  scrape_interval: 1m
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: https
  tls_config:
    insecure_skip_verify: true
  static_configs:
  - targets:
    - 192.168.1.11
    - 192.168.1.12
```

## ALERTMANAGER.YML

```yml
global:
  smtp_from: NAME<EMAIL>
  smtp_smarthost: SMTP_HOST:PORT
  smtp_auth_username: EMAIL
  smtp_auth_password: PASSWORD
  smtp_require_tls: false
  resolve_timeout: 5m

route:
  receiver: email
  group_by: ['alertname']
  group_wait: 1m
  group_interval: 10m
  repeat_interval: 1d
  routes:
    - receiver: email
      matchers:
        - severity =~ 'warning|critical'

receivers:
  - name: email
    email_configs:
      - to: EMAIL

inhibit_rules:
  - source_match:
      severity: critical
    target_match:
      severity: warning
    equal: ['alertname', 'dev', 'instance']
```

## RELOAD CONFIG

curl -X POST http://localhost:9090/-/reload

# MERTICS

## DISK

```
df -B1
Filesystem    1B-blocks       Used          Available       Use%    Mounted on
/dev/sdb1     134681202688    7394095104    120398438400    6%      /data
```

node_filesystem_size_bytes      1B-blocks
node_filesystem_avail_bytes     Available

for `root` filesystem, the `1B-blocks - Used - Available > 0`, the missing space is reserved blocks.
`node_filesystem_free_bytes` represents the free space ignoring the reserved blocks.
so the `used = node_filesystem_size_bytes - node_filesystem_free_bytes`
however in practice you almost always want to use `node_filesystem_avail_bytes` rather than `node_filesystem_free_bytes`
so the `Use% = 100 - (node_filesystem_avail_bytes/node_filesystem_size_bytes * 100)`

## CPU

```js
(1- sum(increase(node_cpu_seconds_total{mode="idle"}[5m])) by (instance) / sum(increase(node_cpu_seconds_total[5m])) by (instance) ) * 100
```

## 内存/磁盘使用率（百分比）

```js
// 内存
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
// 磁盘
(1 - node_filesystem_avail_bytes{mountpoint="/data"}/node_filesystem_size_bytes{mountpoint="/data"}) * 100
```

```yml
Grafana Dashboard:
 - Transform:
    - rename by regex:
      - filter: 
        - Query: A
        - Match: .*"(.*):443".*
        - Replace: $1

  - Standard options:
    - Unit: 
      - Misc: Percent(0.0-1.0)
```

# storage retention

> https://www.robustperception.io/configuring-prometheus-storage-retention/
> https://www.robustperception.io/how-much-disk-space-do-prometheus-blocks-use/


# iostats
https://www.robustperception.io/mapping-iostat-to-the-node-exporters-node_disk_-metrics/


# node exporter

https://github.com/prometheus/node_exporter
https://prometheus.io/docs/guides/node-exporter/

```bash
curl -Ok  https://github.com/prometheus/node_exporter/releases/download/v<VERSION>/node_exporter-<VERSION>.<OS>-<ARCH>.tar.gz

tar xf node_exporter.1.6.1.tgz && rm -f node_exporter.1.6.1.tgz
mv ./node_exporter /usr/local/bin/

groupadd -r prometheus
useradd -r -g prometheus -s /sbin/nologin -M -c "prometheus Daemons" prometheus

tee /usr/lib/systemd/system/node_exporter.service << EOF
[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

[Unit]
Description=node_exporter
After=network.target 
EOF

systemctl enable --now node_exporter

sleep 5
curl -I http://localhost:9100/metrics
```

# blackbox exporter

https://github.com/prometheus/blackbox_exporter

```bash
curl -Ok https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar xf blackbox_exporter-0.25.0.linux-amd64.tar.gz && rm -f blackbox_exporter-0.25.0.linux-amd64.tar.gz
mv ./blackbox_exporter-0.25.0.linux-amd64/blackbox_exporter /usr/local/bin/ && rm -rf ./blackbox_exporter-0.25.0.linux-amd64/

tee /data/scripts/monitor/blackbox_exporter.yml << EOF
modules:
  http_2xx:
    prober: http
    http:
      preferred_ip_protocol: "ip4"
  http_post_2xx:
    prober: http
    http:
      method: POST
EOF

tee /usr/lib/systemd/system/blackbox_exporter.service << EOF
[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/data/scripts/monitor/blackbox_exporter.yml

[Install]
WantedBy=multi-user.target

[Unit]
Description=blackbox_exporter
After=network.target 
EOF

systemctl enable --now blackbox_exporter
sleep 5
curl -I http://localhost:9115/probe
```

# process exporter

https://github.com/ncabatoff/process-exporter
https://github.com/ncabatoff/process-exporter/releases/download/v0.8.2/process-exporter_0.8.2_linux_386.deb
