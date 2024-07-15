# DOCKER COMPOSE
```yml
networks:
  work_net:
    external: true
services:
  gitlab:
    container_name: gitlab
    image: gitlab/gitlab-ce:17
    restart: always
    hostname: 'repo.lindemh-cn.com'
    networks:
      work_net:
    ports:
      - '443:443'
    volumes:
      - ./config:/etc/gitlab
      - ./log:/var/log/gitlab
      - ./data:/var/opt/gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://repo.lindemh-cn.com/gitlab'
        gitlab_rails['omniauth_enabled'] = false
```

# 管理员密码

```bash
grep 'Password:' ./config/initial_root_password
```

# TROUBLESHOOTING

## git fetch-pack: expected shallow list

原因：runner所在主机的git版本太旧，不支持新的语法。

解决方案1：从git1.8升级到git 2.22

解决方案2：修改代码检出方式为git clone

# ERROR: Job failed: prepare environment: exit status 1.

解决方案：移除/home/gitlab-runner/.bash_logout