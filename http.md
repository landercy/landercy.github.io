# Cache-Control

## max-age=n

| 访问次数       | 服务端                                             | 浏览器                     |
| -------------- | -------------------------------------------------- | -------------------------- |
| 首次访问       | 返回200                                | 缓存本地，过期时间`n`秒    |
| 过期前，再访问 | -                                                  |读缓存，不请求服务器 |
| 过期后，再访问 | 资源更新，返回资源，状态码200；资源未更新，返回304 |  304：读缓存                        |


```mermaid
graph TD
start(访问) --> is_1st_visit{是否首次访问}
  is_1st_visit --> |Y|200[服务器返回200]
    200 --> cache[浏览器缓存资源]
    cache --> show[浏览器呈现信息]
  is_1st_visit --> |N|is_expire{是否缓存过期}
    is_expire --> |Y|is_resource_update{是否资源更新}
      is_resource_update --> |Y|200
      is_resource_update --> |N|304[服务器返回304]
        304 --> show
    is_expire --> |N|show
    
```

