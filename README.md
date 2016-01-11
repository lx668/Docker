# 编写一个dockerfile运行sshd 和 lnmp服务,并启动一个博客zblog
#vim dockfile

# vsersion 0.1
# author: lx
# Base images
FROM centos

#Maintainer
MAINTAINER lx 695915099@qq.com

#Commands
#安装nginx、sshd 和 supervisor
RUN rpm -ivh http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
RUN yum install -y nginx openssh-server supervisor mysql
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
RUN mkdir -p /var/run/sshd
RUN mkdir -p /var/log/supervisor
RUN mkdir -p /application
ADD supervisord.conf /etc/supervisor/conf.d/supervisor.conf

#required pkg
ADD Z-BlogPHP.zip /usr/share/nginx/html/
WORKDIR /usr/share/nginx/html/
RUN unzip ./Z-BlogPHP.zip

#install php
RUN yum install zlib-devel libxml2-devel libjpeg-devel freetype-devel libpng-devel gd-devel curl-devel libxslt-devel libmcrypt-devel mhash mhash-devel mcrypt openssl-devel  -y
ADD libiconv-1.14.tar.gz /application
ADD php-5.3.27.tar.gz /application
WORKDIR /application/libiconv-1.14
RUN ./configure --prefix=/usr/local/libiconv && make && make install
WORKDIR /application/php-5.3.27
RUN ./configure --prefix=/application/php5.3.27 --with-mysql=/var/lib/mysql --with-iconv-dir=/usr/local/libiconv --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-safe-mode --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --with-curlwrappers --enable-mbregex --enable-fpm --enable-mbstring --with-mcrypt --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --enable-short-tags --enable-zend-multibyte --enable-static --with-xsl --with-fpm-user=nginx --with-fpm-group=nginx --enable-ftp
RUN ln -s /application/php5.3.27/ /application/php

#Outside Port
EXPOSE 80 22 3306

#supervisor start
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]

#supervisor配置文件内容
[supervisor]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D
process_name=%(program_name)s
autostart=true
stdout_logfile_maxbytes=100MB
studout_logfile_backups=10

[program:nginx]
command=/etc/init.d/nginx
process_name=%(program_name)s
autostart=true
stdout_logfile_maxbytes=100MB
studout_logfile_backups=10

[program:php]
command=/application/php/sbin/php-fpm
process_name=%(program_name)s
autostart=true
stdout_logfile_maxbytes=100MB
studout_logfile_backups=10

[program:mysql]
command=/etc/init.d/mysql start
process_name=%(program_name)s
autostart=true
stdout_logfile_maxbytes=100MB
studout_logfile_backups=10

