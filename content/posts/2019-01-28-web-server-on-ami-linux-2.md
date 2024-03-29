---
title: Web Server on AMI Linux 2
author: admin
type: post
date: 2019-01-28T09:46:20+00:00
url: /2019/web-server-on-ami-linux-2/
thumbnail: images/2015/04/gpu_amazon_ec2_logo.png
categories:
  - Open Source
  - Technologies
  - PHP
  - Web Development
tags:
  - ami linux
  - apache
  - caddy
  - lamp
  - DevOps
  - php

---
Welcome in 2019 &#8211; it&#8217;s time to upgrade out outdated LAMP stack series articles, with new &#8220;How To&#8221; setup basic web server for our stack.

<!--more-->

So&#8230; we have nice and shinny EC2 instance of [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/). On the website, we can read &#8220;Extras in Amazon Linux 2 provides you with bleeding edge software on a stable base of Amazon Linux 2. You no longer need to tradeoff stability for software freshness.&#8221; So&#8230; it should be easier this time. So let&#8217;s see what we have here:

```shell
amazon-linux-extras
  0  ansible2                 available    [ =2.4.2  =2.4.6 ]
  2  httpd_modules            available    [ =1.0 ]
  3  memcached1.5             available    [ =1.5.1 ]
  4  nginx1.12                available    [ =1.12.2 ]
  5  postgresql9.6            available    [ =9.6.6  =9.6.8 ]
  6  postgresql10             available    [ =10 ]
  8  redis4.0                 available    [ =4.0.5  =4.0.10 ]
  9  R3.4                     available    [ =3.4.3 ]
 10  rust1                    available    \
        [ =1.22.1  =1.26.0  =1.26.1  =1.27.2  =1.31.0 ]
 11  vim                      available    [ =8.0 ]
 13  ruby2.4                  available    [ =2.4.2  =2.4.4 ]
  _  php7.2                   available    \
        [ =7.2.0  =7.2.4  =7.2.5  =7.2.8  =7.2.11 ]
 16  php7.1=latest            enabled      [ =7.1.22 ]
  _  lamp-mariadb10.2-php7.2  available    \
        [ =10.2.10_7.2.0  =10.2.10_7.2.4  =10.2.10_7.2.5
          =10.2.10_7.2.8  =10.2.10_7.2.11 ]
 18  libreoffice              available    [ =5.0.6.2_15  =5.3.6.1 ]
 19  gimp                     available    [ =2.8.22 ]
 20  docker=latest            enabled      \
        [ =17.12.1  =18.03.1  =18.06.1 ]
 21  mate-desktop1.x          available    [ =1.19.0  =1.20.0 ]
 22  GraphicsMagick1.3        available    [ =1.3.29 ]
 23  tomcat8.5                available    [ =8.5.31  =8.5.32 ]
 24  epel=latest              enabled      [ =7.11 ]
 25  testing                  available    [ =1.0 ]
 26  ecs                      available    [ =stable ]
 27  corretto8                available    [ =1.8.0_192 ]
 28  firecracker              available    [ =0.11 ]
 29  golang1.11               available    [ =1.11.3 ]
 ```

Pretty slick&#8230; let&#8217;s kick off with EPEL repo, and php7.1  

`amazon-linux-extras install php7.1`

And from now, we can use yum

`yum install php-opcache php-mbstring php-gd php-xml php-pecl-mcrypt`

Let&#8217;s do some configuration php.ini 

```ini
date.timezone = "Australia/Sydney"
expose_php = Off
upload_max_filesize=20M
post_max_size=32M
```

We can just run PHP-FPM with

`systemctl start php-fpm.service`

We also make sure that out server lives in correct Time Zone:

```shell
cd /etc/
sudo rm -rf localtime && sudo ln -s /usr/share/zoneinfo/Australia/Sydney localtime
date
Thu Jan 10 21:06:05 AEDT 2019
```

Looks like in the pretty good spot. Time to install actual web server. You can go with Apache, but I&#8217;m huge fan of [Caddy server](https://caddyserver.com/). It&#8217;s light, it&#8217;s fast and configuration is super easy. You can pull it from the website, compile it locally:

```shell
cd /usr/local/bin
wget ...
chmod a+x caddy
setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy
groupadd caddy
useradd -g caddy --home-dir /var/www/html --no-create-home  --shell /usr/sbin/nologin --system caddy
mkdir /etc/caddy
chown -R root:caddy /etc/caddy
touch /etc/caddy/Caddyfile
chown caddy:caddy /etc/caddy/Caddyfile
chmod 444 /etc/caddy/Caddyfile
./caddy -version
Caddy (untracked dev build) (unofficial)
```

basic config file:

```
nano /etc/caddy/Caddyfile
*:80 {
	root /var/www/html
	gzip
	log /var/log/access.log
	errors /var/log/error.log
	fastcgi / /run/php-fpm/www.sock php
}
```

At this time it&#8217;s worthwhile to upgrade our PHP configuration:

`nano /etc/php-fpm.d/www.conf`

We have to replace user **apache** with **caddy**

```
; RPM: apache user chosen to provide access to the same directories as httpd
user = caddy
; RPM: Keep a group allowed to write in log dir.
group = caddy
```

And. restart php-fpm:

`systemctl restart php-fpm.service`

At this stage we should be able to run working webserver:

`/usr/local/bin/caddy -conf=/etc/caddy/Caddyfile`

Looks cool &#8211; isn&#8217;t it ? Finally we can add service to run our caddy in magical way. Won&#8217;t describe it in detail, just [look here](https://github.com/mholt/caddy/tree/master/dist/init/linux-systemd) for detailed instructions.