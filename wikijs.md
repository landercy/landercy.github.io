```yml
volumes:
  data:
services:
  wikijs:
    container_name: wiki
    image: ghcr.io/requarks/wiki:2
    environment:
      DB_TYPE: sqlite
      DB_FILEPATH: /wiki/data/wiki.db
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - data:/wiki/data
```



## CSS OVERRIDE

```css
/* 使用等宽字体 */
* {
  font-family: "Ubuntu Mono", "Lucida Console"!important
}

/* 新建文档时，隐藏除markdown以外的选项 */
.editor .v-card .row .flex {
  display: none
}
.editor .v-card .row .flex:first-child {
  display: block
}
.editor .v-card .v-slide-group {
  display: none
}
```