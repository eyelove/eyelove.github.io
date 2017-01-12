---
layout: post
fbcomments: false
published: true
title: aws dev instance setting guide
category: linux
tags:
  - linux
  - cli
---

## EC2 Instance Type

- m4.large

## system

- http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/compile-software.html

```
sudo yum -y update
sudo yum -y groupinstall "Development Tools"

sudo passwd
```

## mariadb v10.0

sudo vim /etc/yum.repos.d/mariadb.repo

```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.0/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

```
sudo yum -y install MariaDB-server MariaDB-client
sudo cp -a /usr/share/mysql/my-huge.cnf /etc/my.cnf.d/
sudo mkdir /home/mysqldb
```

sudo vim /etc/my.cnf.d/my-huge.cnf

```
[client]
default-character-set = utf8
[mysqld]
datadir         = /home/mysqldb
skip-name-resolve
default-storage-engine = INNODB
init_connect = "SET collation_connection = utf8_general_ci"
init_connect = "SET NAMES utf8"
character-set-server=utf8
collation-server = utf8_general_ci
skip-character-set-client-handshake
lower_case_table_names=1
max_connect_errors = 1844674407370954751
connect_timeout = 20
slave_net_timeout = 30
#log-bin=mysql-bin
thread_handling = pool-of-threads
[mysql]
default-character-set=utf8
```

```
sudo mysql_install_db
sudo chown mysql: -R /home/mysqldb
sudo service mysql start
sudo mysqladmin -u root password 'new-password'
```

### Security

mysql -u root -p

```sql
MariaDB [(none)]> use mysql
MariaDB [mysql]> delete from user where password = '';
```

### Tunning 

```
cd ~
wget http://mysqltuner.pl/ -O mysqltuner.pl
chmod +x mysqltuner.pl
perl mysqltuner.pl
```

## mongodb v3.2

sudo vim /etc/yum.repos.d/mongodb-org-3.2.repo

```
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.2/x86_64/
gpgcheck=0
enabled=1
```

```
sudo yum -y install mongodb-org mongodb-org-mongos mongodb-org-server mongodb-org-shell mongodb-org-tools
```

sudo vim /etc/mongod.conf

```
storage:
  dbPath: /home/mongo
```

```
sudo mkdir -p /home/mongo
sudo chown mongod.mongod /home/mongo
sudo service mongod start
```

## redis

```
sudo sysctl vm.overcommit_memory=1

cd /usr/local/src/
sudo wget http://download.redis.io/redis-stable.tar.gz
sudo tar xvzf redis-stable.tar.gz
cd redis-stable
```

gcc 나 jemalloc 를 못찾는 오류가 발생할 경우 yum으로 "Development Tools"을 설치후 아래 명령어 실행 후 다시 make한다.

```
sudo make distclean 
```

```
sudo make

sudo cp src/redis-server /usr/local/bin/
sudo cp src/redis-cli /usr/local/bin/
sudo cp src/redis-sentinel /usr/local/bin/
```

### port conf

- sentinel 47000
- redis 47001 , 47002, 47003

```
sudo mkdir /etc/redis
sudo mkdir /var/redis
sudo cp utils/redis_init_script /etc/init.d/redis_sentinel
sudo cp utils/redis_init_script /etc/init.d/redis_47001
```

sudo vim /etc/init.d/redis_47001

```sh
#!/bin/sh
#
# chkconfig: - 50 50
# description: redis 47001 server daemon
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=47001
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
...
```


sudo vim /etc/init.d/redis_sentinel

```sh
#!/bin/sh
#
# chkconfig: - 40 50
# description: redis sentinel daemon
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=47000
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF --sentinel
        fi
...
```

```
sudo cp sentinel.conf /etc/redis/47000.conf
sudo cp redis.conf /etc/redis/47001.conf
```

sudo vim /etc/redis/47001.conf

```
port 47001
pidfile /var/run/redis_47001.pid
logfile "/var/log/redis_47001.log"
dir "/var/redis/47001"
```

sudo vim /etc/redis/47000.conf

```
port 47000

daemonize yes
supervised no
pidfile "/var/run/redis_47000.pid"

sentinel monitor redis47001 127.0.0.1 47001 1
sentinel monitor redis47002 127.0.0.1 47002 1
sentinel monitor redis47003 127.0.0.1 47003 1

sentinel down-after-milliseconds redis47001 5000
sentinel down-after-milliseconds redis47002 5000
sentinel down-after-milliseconds redis47003 5000

sentinel parallel-syncs redis47001 1
sentinel parallel-syncs redis47002 1
sentinel parallel-syncs redis47003 1

sentinel failover-timeout redis47001 5000
sentinel failover-timeout redis47002 5000
sentinel failover-timeout redis47003 5000
```

```
sudo mkdir /var/redis/47001
sudo mkdir /var/redis/47002
sudo mkdir /var/redis/47003
```

- redis pid list
  - /var/run/redis_47000.pid
  - /var/run/redis_47001.pid
  - /var/run/redis_47002.pid
  - /var/run/redis_47003.pid

```
sudo chkconfig --add redis_sentinel
sudo chkconfig --add redis_47001
sudo chkconfig --add redis_47002
sudo chkconfig --add redis_47003
```

## nginx + php7
```
sudo yum install openssl-devel

cd /usr/local/src
sudo rpm -Uvh ftp://ftp.scientificlinux.org/linux/scientific/6.4/x86_64/updates/fastbugs/scl-utils-20120927-8.el6.x86_64.rpm
sudo wget http://mirrors.mediatemple.net/remi/enterprise/remi-release-6.rpm
sudo rpm -ivh remi-release-6.rpm

sudo yum -y install nginx
sudo yum -y --enablerepo=remi install php70-php php70-php-fpm php70-php-cli php70-php-common  php70-php-json php70-php-mbstring php70-php-mcrypt php70-php-mysqlnd php70-php-opcache php70-php-pear php70-php-xml php70-php-devel php70-php-pecl-memcached php70-php-pecl-msgpack php70-php-pecl-redis


sudo mkdir -p /home/httpd/logs/
sudo mkdir -p /home/httpd/logs/nginx
sudo mkdir -p /home/httpd/logs/php-fpm
sudo mkdir -p /home/www/
sudo chown ec2-user.ec2-user /home/www
```

- mongodb driver가 정상적으로 로드되지 않아 아래와 같이 직접 설치한다.

```
source /opt/remi/php70/enable

sudo /opt/remi/php70/root/usr/bin/pecl install mongodb

sudo sh -c "echo 'extension=mongodb.so' > /etc/opt/remi/php70/php.d/50-mongodb.ini"
```

#### test setting

```
mkdir /home/www/ssp
```

sudo vim /etc/nginx/nginx.conf

```
user nobody;
error_log /home/httpd/logs/nginx/error.log;
access_log  /home/httpd/logs/access.log  main;
```

sudo vim /etc/nginx/conf.d/dev.conf

```
server {
    listen 80;
    server_name *.compute.amazonaws.com;
    root /home/www/webroot/ssp/public;
    index index.php index.html;

    access_log /home/httpd/logs/ssp_access.log main;
    error_log /home/httpd/logs/ssp_error.log;

    location / {
        try_files $uri $uri/ /index.php;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php7-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_script_name;
        fastcgi_param CI_ENV "testing";
        include /etc/nginx/fastcgi_params;
    }
}
```

sudo vim /etc/opt/remi/php70/php-fpm.conf

```
error_log = /home/httpd/logs/php-fpm/error.log
```

sudo vim /etc/opt/remi/php70/php.ini

```
short_open_tag = On
expose_php = Off
error_reporting = E_ALL
display_startup_errors = On
track_errors = On
html_errors = Off
error_log = /home/httpd/logs/php-fpm/php_errors.log
date.timezone = UTC
```

sudo vim /etc/opt/remi/php70/php-fpm.d/www.conf

```
listen = /var/run/php7-fpm.sock
listen.owner = nobody
listen.group = nobody
listen.mode = 0660
```


```
sudo service php70-php-fpm start
sudo service nginx start
```


##### Fileupload 500 Error

```
open() "/var/lib/nginx/tmp/client_body/0000000002" failed (13: Permission denied)
chown -R nobody.nobody /var/lib/nginx
```
- http://nishal-tech.blogspot.kr/2013/06/nginx-13-permission-denied-while.html

## boot on

```
sudo chkconfig mongod on
sudo chkconfig mysql on
sudo chkconfig nginx on
sudo chkconfig php70-php-fpm on
sudo chkconfig redis_sentinel on
sudo chkconfig redis_47001 on
sudo chkconfig redis_47002 on
sudo chkconfig redis_47003 on
```

## CodeIgniter (web project)

```
cd /home/www
wget https://github.com/bcit-ci/CodeIgniter/archive/3.0.6.tar.gz
tar xvfz 3.0.6.tar.gz
ln -s -d /home/www/CodeIgniter-3.0.6 /home/www/webroot
cd /home/www/webroot
cp -r application new_app
mkdir new_app/public
cp index.php new_app/public/index.php
```

vim ssp/public/index.php

```
        $public_path = dirname(__FILE__);
        $site_root = dirname(dirname($public_path));

        $system_path = $site_root.'/system';

        $application_folder = dirname($public_path);
```
