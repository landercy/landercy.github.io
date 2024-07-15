# DOCKER COMPOSE

```yml
networks:
  work_net:
    external: true
services:
  grafana_monitor:
    container_name: grafana_monitor
    image: grafana/grafana:latest
    restart: unless-stopped
    networks:
      - work_net
    ports:
      - '3000:3000'
    volumes:
      - ./grafana:/var/lib/grafana
    environment:
      GF_SERVER_ROOT_URL: https://10.81.104.11/monitor
      GF_SERVER_SERVE_FROM_SUB_PATH: true
```

```nginx
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

upstream grafana_monitor {
  server 127.0.0.1:3000;
}

server {
  location /monitor/ {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://grafana_monitor;
  }
  location /monitor/api/live/ {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $http_host;
    proxy_pass http://grafana_monitor;
  }
}