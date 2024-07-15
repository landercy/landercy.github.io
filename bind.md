# Bind - DNS

## Client

```sh
vim /etc/resolv.conf
```

>search lan
>
>nameserver 10.23.0.10

```sh
chattr +i /etc/resolv.conf 
```

## Installation

```sh
yum install -y bind
rpm -qa bind
```

## Configuration

```sh
vi /etc/named.conf
```

```c
options {
//listen-on port 53 { 127.0.0.1; };
  listen-on port 53 { 10.23.0.10; };
//listen-on-v6 port 53 { ::1; };
  directory   "/var/named";
  dump-file   "/var/named/data/cache_dump.db";
  statistics-file "/var/named/data/named_stats.txt";
  memstatistics-file "/var/named/data/named_mem_stats.txt";
  recursing-file  "/var/named/data/named.recursing";
  secroots-file   "/var/named/data/named.secroots";
//allow-query     { localhost; };
  allow-query     { 10.23.0.0/24; };
  
  recursion yes;

  forward only;
  forwarders { 223.5.5.5; };
  
//dnssec-enable yes;
  dnssec-enable no;
//dnssec-validation yes;
  dnssec-validation no;
```

```sh
tee >> /etc/named.rfc1912.zones << EOF
zone "lan" IN {
  type master;
  file "lan.zone";
  allow-update { 10.23.0.10; };
};
EOF
```

```sh
cd /var/named
cp named.empty lan.zone
chown root:named lan.zone
vim lan.zone
```

```sh
$TTL 3H
@       IN SOA  @ landercy@aliyun.com (
                                        01      ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       10.23.0.10

dns  A 10.23.0.10
vm10 A 10.23.0.10
vm11 A 10.23.0.11
vm12 A 10.23.0.12
vm13 A 10.23.0.13
vm14 A 10.23.0.14
vm15 A 10.23.0.15
vm16 A 10.23.0.16
vm17 A 10.23.0.17
vm18 A 10.23.0.18
vm19 A 10.23.0.19
vm20 A 10.23.0.20
```

>第2行的 `@` 是$ORIGIN，如果未在之间声明`$ORIGIN host.vm`，则`@`取name.conf里的zone
>
>SOA <授权主机> <管理邮箱>
>
>由于`@`是保留关键字，因此邮箱中的`@`使用`.`代替
>
>serial: slave进行同步时会比较该字段，如数值比自己大则更新。一般可设置为 “年月日+2位修改次序”
>
>refresh: slave同步时间间隔
>
>retry: slave同步失败后再次尝试的时间间隔
>
>expire: 当slave长时间无法与master取得联系，slave数据过期时间
>
>minimum: 最小TTL值，当未声明`$TTL`$时，以此值为准
>
>格式：
>
>定义记录项（Entry）时，若首位是空格，则将被视为是上一个项的内容

```sh
named-checkconf
systemctl start named
systemctl status named
netstat -lnput | grep 53
ping dns.vm
ping n10.vm
dig dns.host.com @192.168.56.10 +short
```

## Troubleshooting

### dig dns.host.com @192.168.56.10 +short 无输出

> 检查 /var/named/vm.zone 权限
>
> 默认为 640 root:root 
>
> 改为 640 root:named
