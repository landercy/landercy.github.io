# DOCKER COMPOSE

```yml
networks:
  work_net:
    external: true
services:
  zentao:
    image: hub.zentao.net/app/zentao:20.2.0
    container_name: zentao
    networks:
      work_net:
    ports:
      - '8080:80'
    volumes:
      - ./data:/data
    environment:
      - MYSQL_INTERNAL=false
      - ZT_MYSQL_HOST=mysql57
      - ZT_MYSQL_PORT=3306
      - ZT_MYSQL_USER=zentao
      - ZT_MYSQL_PASSWORD=zentao_password
      - ZT_MYSQL_DB=zentao
      - PHP_MAX_EXECUTION_TIME=120
      - PHP_MEMORY_LIMIT=512M
      - PHP_POST_MAX_SIZE=128M
      - PHP_UPLOAD_MAX_FILESIZE=128M
      - LDAP_ENABLED=false
      - APP_DEFAULT_PORT=80
      - APP_DOMAIN=localhost
      - PROTOCOL_TYPE=http
      - IS_CONTAINER=true
      - LINK_GIT=false
      - LINK_CI=false
```

# 补丁

## 修复无法指派外部人员的问题

/apps/zentao/module/user/model.php:269

注释掉人员类别判断

```php
public function getPairs() {
...
 266  $users = $this->dao->select($fields)->from(TABLE_USER)
 267    ->where('1=1')
 268    ->beginIF(strpos($params, 'nodeleted') !== false || empty($this->config->user->showDeleted))->andWhere('deleted')->eq('0')->fi()
 269    //->beginIF(strpos($params, 'all') === false)->andWhere('type')->eq($type)->fi()
 270    ->beginIF($accounts)->andWhere('account')->in($accounts)->fi()
 271    ->beginIF($this->config->vision && $this->app->rawModule !== 'kanban')->andWhere("FIND_IN_SET('{$this->config->vision}', visions)")->fi()
 272    ->orderBy($orderBy)
 273    ->beginIF($maxCount)->limit($maxCount)->fi()
 274    ->fetchAll($keyField);
 ...
}
```