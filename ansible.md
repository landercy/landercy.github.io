```bash
ans -u www -i hosts -m ping all

ssh-keygen -f ~/.ssh/known_hosts -R HOST_IP
ans -u www -i ./hosts -m shell -a "df -h|grep -E '/$|/data$'" all

```

```bash
# 查看配置文件
ans --version
```

```ini
# 不校验域名
[defaults]
host_key_checking= False
```
