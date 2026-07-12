---
title: NTTU 伺服器原理 WriteUp
date: 2024-08-06 06:28:35
category: csie
tags: 
    - server
    - csie
---

1. Ansible
2. SELinux

## DNS server

### Intro (2023.09.20)

1. SELinux review
    + `ls -lZ`
    + `semanage`
    + `setenforce`
    + `chcon`
    + `firewalld`

2. 名詞解釋

    + DNS : Domain name service
    + FQDN : Fully Qualified Domain Name
    + Domain
    + Subdomain
    + **Zone** : Depand on Domain manager

3. client
    + `/etc/nsswitch.conf` (Name Sever Switch)
    + `/etc/hosts` : Test usage , contains IP host names and addresses for the local host and other hosts in the Internet network
    + `/etc/resolv.conf` : DNS

4. Command
    + `host`
    + `nslooup`

5. DNS Port
    + `53` tcp , udp

6. DNS resource record
    + `host -v -t A example.com`
        -> `example.com. 86400 IN A 172.25.254.254`
    + A : Ipv4
    + AAAA : Ipv6
    + SOA
    + NS : Name Server
    + MX : Mail Exchange
    + CNAME : Cononical Name
    + TXT : Text
    + SRV : Service

[RedHat Document](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/9/html/managing_networking_infrastructure_services/assembly_setting-up-and-configuring-a-bind-dns-server_networking-infrastructure-services)

### Bind_Named (2023.09.27)

#### Directery

+ `/var/named/` 主要目錄
+ `/var/named/slaves/` secondary zones 使用
+ `/var/named/dynamic/` dynamic DNS (DDNS) zones 或 DNSSEC keys.
+ `/var/named/data/` 統計與除錯檔案

#### Ways

+ 正查

```bash=
$dnf install bind-chroot
$vim /etc/named.conf : # setup zone
    # add ip addr and change allow to any
    # **Add Text** :
        zone "LLL.tw" IN {
            type master;
            file "LLL.tw.zone";
        }
$cd /var/named
$cp named.empty LLL.tw.zone
$vim LLL.tw.zone
```

+ `LLL.tw.zone`

```txt=
$TTL 86400
$ORIGIN LLL.tw.
@       IN      SOA     dns1.example.com. admin.example.com.( #域名伺服器的名稱 管理者郵箱 
                       2022010101  ; Serial
                       3600        ; Refresh
                       1800        ; Retry
                       604800      ; Expire
                       86400       ; Minimum TTL
);

        NS     dns1.example.com.
www     IN      A    172.25.250.10
```

$systemctl restart named
$chown root.named LLL.tw.zone

```bash=
$firewall-cmd --permanent --add-service=dns
$firewall-cmd --reload

$nmcli con mod "Net1" ipv4.dns 172.25.250.10
$nmcli con reload

# To other machine
$dig @172.25.250.10 dns.servera.tw
```

+ Command history

    ![image](https://hackmd.io/_uploads/S16vfJEZp.png)
+ 反解

```txt=
$TTL 86400
@       IN      SOA     ns1.example.com. admin.example.com. (
                       2022010101  ; Serial
                       3600        ; Refresh
                       1800        ; Retry
                       604800      ; Expire
                       86400       ; Minimum TTL
);

@       IN      NS      ns1.example.com.

1       IN      PTR     example.com.

```

### Unbound

+ `/etc/unbound/unbound.conf`

```txt=
local-zone: "example.com." static
local-data: "example.com. IN A 192.168.1.100"
```

```bash=
$sudo chown -R unbound:unbound /var/lib/unbound
$sudo systemctl restart unbound
```

## Web server

### HTTPd

#### Custom web page

+ Basic config`/etc/httpd/conf/http.conf`
+ custom `*.conf` add in `/etc/httpd/conf.d`

    ```planttxt
    <Directory "/var/www/html/user">
        AllowOverride None
        Require all granted
    </Directory>


    <VirtualHost *:80>
        DocumentRoot "/var/www/html/user"
        ServerName www.user.tw
        ServerAdmin lll@nttu.edu.tw
        ErrorLog "logs/user_error_log"
        CustomLog "logs/user_cos_log" combined
    </VirtualHost>
    ```

+ and add dir in `/var/www/html/`

### nginx

+ `/etc/nginx/nginx.conf`
+ `/etc/nginx/conf.d`
+ Add virtual server for page `edit *.conf`

```conf=
server{
    listen 80;
    listen [::]:80
    server_name www.jjli.tw; # virtual host
    root /usr/share/nginx/html/<user>; # virtual host root direction.
}
```

+ `/usr/share/nginx/html`

### HTTPS

+ **httpd security cerify**
![image](https://hackmd.io/_uploads/BJtoTV2DT.png)

## Cache server

### Varnish

+ Modify service

```bash=
$systemctl cat varnish #get ExecStart and modify in http_port.conf
$cd /etc/systemd/system
$mkdir varnish.service.d
$vim varnish.service.d/http_port.conf
```

+ `varnish.service.d/http_port.conf` daemon 參數調整

```txt
[service]
ExecStart=
ExecStart=/usr/
```

+ start service

```bash=
$systemctl daemon-reload
$systemctl restart varnishd
$firewall-cmd --permanent --add-service=http
$firewall-cmd --reload
```

+ 快取配置
`/etc/varnish/default.vc`

## Proxy server

+ 正向 & 反向代理

![image](https://hackmd.io/_uploads/S1hykuGOp.png)

+ 負載平衡

```bash=
$vim /etc/haproxy/*.conf # edit frontend and backend
# vim
frontend ll
    bind *:80
    default_backend lll_servers

backend lll_servers
    balance <roundrobin,source>
    server weba 172.25.250.10:80 check inter 10s
    server webb 172.25.250.11:80 check inter 10s

```

```bash
$firewalll-cmd --permanent --add-service=http # open firewall for service.
```

+ Https 解密

![截圖 2023-12-30 16.29.44](https://hackmd.io/_uploads/BJB-rIav6.png)

+ 統計

![截圖 2023-12-30 16.40.56](https://hackmd.io/_uploads/B1KsPLTDT.png)

```bash=
stats uri /<url> # 自己指定的網頁名稱
stats auth username:passwd
```

### Varnish + haproxy

+ varnish 在haproxy後 （ 或在haproxy前 ）
![image](https://hackmd.io/_uploads/By4f9Lpvp.png)

### 小結

![image](https://hackmd.io/_uploads/HydC9Lpwp.png)

## DataBase

+ MariaDB
+ phpMyAdmin

```bash=
$sudo mysql_secure_installation # setup root passwd
$sudo mysql -u root -p

# mariadb command
CREATE DATABASE your_database_name;
CREATE USER 'your_username'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON your_database_name.* TO 'your_username'@'localhost';
FLUSH PRIVILEGES;

$sudo systemctl restart mariadb
```
