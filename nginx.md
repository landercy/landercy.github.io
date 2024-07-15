# 配置

## SSL

```nginx
server {
    server_name     _;
    listen          443 ssl default_server;

    root            /usr/share/nginx/html;
    index           index.html;

    ssl_certificate             "/data/apps/nginx/certs/xxx/server.crt";
    ssl_certificate_key         "/data/apps/nginx/certs/xxx/server.key";
    ssl_protocols               TLSv1.2;
    ssl_ciphers                 ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:AES256-GCM-SHA384;
    ssl_prefer_server_ciphers   on;

    location / {
    }
}
```

## PHP PROXY

```nginx
location / {
  try_files $uri /index.php$is_args$args;
}

location ~ \.php$ {
  fastcgi_pass   127.0.0.1:9000;
  fastcgi_index  index.php;
  fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  fastcgi_param  PATH_INFO $fastcgi_path_info;
  include        fastcgi_params;
}

# example using alias
location /web {
  alias /dir/project/public;
  index index.php;  
  location ~ .+\.php {
    fastcgi_pass   127.0.0.1:9000;
    # 设置SCRIPT_FILENAME和PATH_INFO变量
    # 如果不设置此行
    # $fastcgi_script_name 值会是 /web/index.php
    # SCRIPT_FILENAME 值会是 /dir/project/public/web/index.php
    # 显然该文件不存在，因此访问时会报 FastCGI sent in stderr: "Primary script unknown" while reading response header from upstream
    fastcgi_split_path_info ^/web(.+\.php)(.*)$;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    fastcgi_param  PATH_INFO $fastcgi_path_info;
    include        fastcgi_params;
  }
}
```

## PROXY HEADER
```nginx
location / {
    proxy_set_header Host               $host;
    proxy_set_header Remote_Addr        $remote_addr;
    proxy_set_header X-Real-IP          $remote_addr;
    proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto  $scheme;
    proxy_pass http://127.0.0.1:8080;
}
```

## REDIRECT

```nginx
# 301
rewrite ^/(.*)$ http://xxx.com/$1 permanent;
# 302
rewrite ^/(.*)$ http://xxx.com/$1 redirect;
```

## CORS

```nginx
add_header Access-Control-Allow-Origin '*';
add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';

if ($request_method = 'OPTIONS') {
    return 204;
}
```

## SHOW INDEX

```nginx
location / {
  autoindex on;
  autoindex_exact_size off;
  autoindex_localtime off;
}
```

## BLOCK ACCESS

```nginx
# no access to configuration files
location ~ \.(env|ini|cnf|conf|config|xml|yml|yaml|properties|ht|htaccess)$ {
  deny all;
}

# no access to workspace meta files
location ~ \.(svn|git|hg|bzr|idea|iml|vscode|project|classpath|metadata)$ {
  deny all;
}

# no access to bash script file
location ~ \.(sh|bash|zsh|shell|script|scripts)$ {
  deny all;
}

# no access to source file
location ~ \.(c|h|go|asp|aspx|jsp|class|php|php3|php4|php5|phtml|pl|perl|cgi|fcgi|py|pyc|pyo|rb|rhtml|lua)$ {
  deny all;
}

# no access to sensitive files
location ~ \.(sql|db|log|bak)$ {
  deny all;
}

# no access to description files
location ~ (LICENSE|README.md)$ {
  deny all;
}

# add cache to static files
location ~ \.(js|css|jpg|jpeg|png|svg|ico|ttf|otf|woff|woff2)$ {
  expires 30d;
  access_log off;
}
```

# PRINT VAR

```nginx
location /aa/ {
  default_type text/html;
  return 200 "AA";
}
```

# PROXY PASS 移除前缀

```nginx
# 访问 /aa/bb
# 后端 /bb
location ~ /aa/(.*) {
  proxy_pass http://127.0.0.1:8001/$1$is_args$args;
}
```

# 证书

## 自签

```bash
# C: Country
# ST: State/ Province
# L: Location/ City
# O: Organization/ Company
# OU: Orgainzational Unit/ Department
# CN: Common Name/ Hostname/ IP

dir="/data/apps/nginx/certs/default"
if [ ! -d $dir ]
then
mkdir -p $dir
fi
cd $dir
openssl genrsa -des3 -passout pass:1234 -out server.key 2048
openssl req -new -passin pass:1234 -subj "/C=CN/ST=Fujian/L=Xiamen/O=Linde/OU=IT/CN=$(hostname -i)" -key server.key -out server.csr
mv server.key server.key.org
openssl rsa -passin pass:1234 -in server.key.org -out server.key
openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
rm -f server.csr server.key.org && ls -l
openssl x509 -in server.crt -noout -dates|grep notAfter|cut -c10-29

# 查看证书过期时间
openssl x509 -in server.crt -noout -dates

# 查看证书域名
openssl x509 -in server.crt -noout -text|grep "Subject:"
```

## 证书链

```bash
openssl s_client -connect example.com:443 -showcerts | grep "[si]:"
```

## 格式转换

```bash
# PKCS1 转 PCKS8
openssl pkcs8 -topk8 -nocrypt -in server.key -out server.pem
# PKCS8 转 PCKS1
openssl rsa -in server.pem -out server.key
```

```
# 查看每个证书的Issuer与下一个证书的Subjeect是否匹配
 0 s:CN = *.lindemh-cn.com
   i:C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Domain Validation Secure Server CA
 1 s:C = GB, ST = Greater Manchester, L = Salford, O = Sectigo Limited, CN = Sectigo RSA Domain Validation Secure Server CA
   i:C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
 2 s:C = US, ST = New Jersey, L = Jersey City, O = The USERTRUST Network, CN = USERTrust RSA Certification Authority
   i:C = GB, ST = Greater Manchester, L = Salford, O = Comodo CA Limited, CN = AAA Certificate Services
 3 s:C = GB, ST = Greater Manchester, L = Salford, O = Comodo CA Limited, CN = AAA Certificate Services
   i:C = GB, ST = Greater Manchester, L = Salford, O = Comodo CA Limited, CN = AAA Certificate Services
```

## 信任证书
echo -n | openssl s_client -showcerts -connect registry.local:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> /etc/ssl/certs/ca-certificates.crt
update-ca-certificates


# 问题解决方案

## Https请求，使用http进行代理后，程序获取协议和域名错误的问题

nginx 配置了 proxy

```nginx
location / {
  proxy_pass  http://127.0.0.1:8088;
}
```

JSP 程序获取请求信息，拼接静态资源地址

```java
<%
String path = request.getContextPath();
String basePath = request.getScheme() + "://" + 
  								request.getServerName() + ":" + 
  								request.getServerPort() + path + "/";

<base href="<%=basePath%>">

<link rel="stylesheet" href="static/xxx/xxx.css" />
```

浏览器页面访问地址

> https://hostname/xxx

浏览器资源访问地址，资源域名错误

> http://127.0.0.1:8088/static/xxx/xxx.css:

修改nginx proxy

```nginx
location / {
  proxy_pass  				http://127.0.0.1:8088;
  proxy_set_header		Host				$host;
}
```

浏览器资源访问地址，资源域名正常，但协议错误

> http://hostname/static/xxx/xxx.css:

修改nginx proxy

```nginx
location / {
  proxy_pass  				http://127.0.0.1:8088;
  proxy_set_header		Host										$host;
  proxy_set_header		X-Forwarded-Proto				$scheme;
}
```

浏览器资源访问地址，协议依然错误

> GET http://hostname/static/xxx/xxx.css:

经查询，需要配置Tomcat server.xml，在Engine 模块下配置一个 Valve

```xml
<Valve className="org.apache.catalina.valves.RemoteIpValve"  
remoteIpHeader="X-Forwarded-For"  
protocolHeader="X-Forwarded-Proto"  
protocolHeaderHttpsValue="https"/> 
```

浏览器资源访问地址，协议正确

> https://hostname/static/xxx/xxx.css:

A --> B --> C --> D

remote_addr 为上一级ip，即 D的remote_addr 为C，B的remote_addr是A

x-real-ip设置为 remote_addr，所以D的x-real-ip是C的remote_addr，即B

x-forward-for 为上级前(不含)的ip，即D的x-forward-for是 A, B

尝试 将C的x-real-ip 设置为B传递的x-real-ip，意图拿到A，但是无法在C中获取到B的real-ip，未果


## 非root监听443端口

nginx.service中

```ini
User=www
Group=www
PIDFile=/data/apps/nginx/PID
```

修改pid文件位置，注意与nginx.conf一致

```bash
setcap cap_net_bind_service=+eip /usr/sbin/nginx
```

nginx version: nginx/1.14.0 (Ubuntu)
alias 不写最后的“/”，访问时会报403，不是权限的问题


## 检测到SHA-1密码套件

```nginx
ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:AES256-GCM-SHA384;
```

```bash

nmap --script ssl-enum-ciphers -p 443 diagnosis.kion-cn.com

Starting Nmap 6.40 ( http://nmap.org ) at 2022-10-14 10:56 CST
Nmap scan report for e-qa.kion-cn.com (127.0.0.1)
Host is up (0.000054s latency).
rDNS record for 127.0.0.1: localhost
PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.2:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 - strong
|       TLS_RSA_WITH_AES_256_GCM_SHA384 - strong
|     compressors:
|       NULL
|_  least strength: strong

Nmap done: 1 IP address (1 host up) scanned in 0.67 seconds

# BREAK VS LAST IN REWRITE

A       : 1.html -> 2.html -> 3.html -> b.html
B[break]: 1.html -> 2.html
C[last] : 1.html -> 2.html -> a.html

```nginx
# A
server {
  listen 80;
  root   /data/web;
  index  index.html;

  location / {
    rewrite /1.html /2.html;
    rewrite /2.html /3.html;
  }

  location /2.html {
    rewrite . /a.html;
  }

  location /3.html {
    rewrite . /b.html;
  }
}
# B
server {
  listen 80;
  root   /data/web;
  index  index.html;

  location / {
    rewrite /1.html /2.html break;
    rewrite /2.html /3.html;
  }

  location /2.html {
    rewrite . /a.html;
  }

  location /3.html {
    rewrite . /b.html;
  }
}
# C
server {
  listen 80;
  root   /data/web;
  index  index.html;

  location / {
    rewrite /1.html /2.html last;
    rewrite /2.html /3.html;
  }

  location /2.html {
    rewrite . /a.html;
  }

  location /3.html {
    rewrite . /b.html;
  }
}
```

# TROUBLE SHOOTING

## net::ERR_CONTENT_LENGTH_MISMATCH 200 (OK)
error.log
open() "/var/cache/nginx/proxy_temp/6/01/0000000016" failed (13: Permission denied) 
chown -R www:www /var/cache/nginx


## net::ERR_CONTENT_LENGTH_MISMATCH 206 (Partial Content) net::ERR_INCOMPLETE_CHUNKED_ENCODING 200 (OK)
CSS/JS文件直接访问正常，但是网页加载时console报错
nginx error log报：
> open() "/var/cache/nginx/proxy_temp/2/12/0000000122" failed (13: Permission denied) while reading upstream
chown -R www:www /var/cache/nginx 
赋权后恢复正常