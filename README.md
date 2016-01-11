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
ADD supervisord.conf /etc/supervisor/conf.d/supervisor.conf

#Outside Port
EXPOSE 80 22 3306
CMD ["/usr/bin/supervisord"]

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

[program:mysql]
command=/etc/init.d/mysql start
process_name=%(program_name)s
autostart=true
stdout_logfile_maxbytes=100MB
studout_logfile_backups=10

