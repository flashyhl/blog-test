---
layout: page
title: Ansible安装部署Blog的基础环境 
permalink: /ansible-install-env/
---
 
-------
安装前准备：  
下载 Nginx：  
\# wget http://nginx.org/download/nginx-1.8.1.tar.gz -P /usr/local/src/nginx-1.8.1.tar.gz  
下载 Rudy：
\# wget https://ruby.taobao.org/mirrors/ruby/2.3/ruby-2.3.0.tar.gz -P /usr/local/src/  
nginx 配置文件如下：  
vim /usr/local/src/nginx.conf  

```
user  www www;
worker_processes  4;
worker_rlimit_nofile 51200;
#error_log  /etc/nginx/logs/error.log warn;
pid        /etc/nginx/run/nginx.pid;
events {
    use epoll;
    worker_connections  51200;
}
http{
    include mime.types;
    default_type application/octet-stream;
    sendfile on;

    tcp_nopush      on;
    tcp_nodelay     on;
    server_tokens   off;

    server_names_hash_bucket_size   128;
    client_header_buffer_size      512k;
    client_body_buffer_size        512k;
    client_max_body_size            50m;
    large_client_header_buffers  4 512k;

    send_timeout 500;
    client_header_timeout           500;
    client_body_timeout             500;

    connection_pool_size 256;
    request_pool_size 4k;
    output_buffers 4 32k;
    postpone_output 1460;

    gzip on;
    gzip_vary on;
    gzip_static on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 6;
    gzip_types text/plain application/x-javascript application/javascript text/css application/xml text/javascript;
    gzip_disable "MSIE [1-6]\.";

    fastcgi_intercept_errors on;
    keepalive_timeout  120;
    proxy_buffering on;
    proxy_buffer_size 512k;
    proxy_buffers   32 256k;
    proxy_busy_buffers_size 512k;

    proxy_ignore_client_abort off;
    proxy_intercept_errors    on;
    proxy_redirect            off;
    proxy_set_header          X-Forwarded-For $remote_addr;
    proxy_connect_timeout     500;
    proxy_send_timeout        500;
    proxy_read_timeout        500;




    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 16 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    fastcgi_temp_path /dev/shm;
    log_format lan_format  '$remote_addr - $remote_user [$time_local] "$request" '
                                    '$status $body_bytes_sent "$request_time" "$http_referer" '
                                    '"$http_user_agent" $http_x_forwarded_for ';

        log_format  main  '$remote_addr - $remote_user [$time_local] [$request_time $upstream_response_time] "\$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$http_cookie" ';

    #access_log      logs/access.log;
    include          conf.d/*.conf;
}
```
Nginx服务启动配置文件如下：  
```  
#!/bin/bash
#
# Startup script for Nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# pidfile:     /etc/nginx/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/etc/nginx/nginx.conf"
 
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
#### 一、Nginx的安装
1、创建Ansible的安装目录  

\# mkdir /ansible-test/nginx  

\# cd /ansible-test/nginx  


2、创建配置目录  

\# mkdir -p roles/{common,install}/{handlers,files,meta,tasks,templates,vars}  
  
```
结构如下：
    .
    └── roles
        ├── common
        │   ├── files
        │   ├── handlers
        │   ├── meta
        │   ├── tasks
        │   ├── templates
        │   └── vars
        └── install
            ├── files
            ├── handlers
            ├── meta
            ├── tasks
            ├── templates
            └── vars   
```                    
说明如下：                
* roles 目录下有两个角色，common为一些准备操作，install 为安装nginx 的操作。
* 每个角色下有几个目录：
* handlers :当发生改变时要执行的操作，通常用在配置文件发生改变，重启服务
* files :为安装时用到的一些文件
* meta :为说明信息，说明角色依赖等信息
* tasks :核心的配置文件
* templates :通常存一些配置文件，启动脚本等模板文件
* vars : 定义的变量

3、文件处理
common目录下操作：
\# cd /ansible-test/nginx/roles/common/tasks  

\# vim main.yml  

```
- name: Install initialization require software
  yum: name=\{\{ item \}\} state=installed
  with_items:
    - gcc
    - gcc-c++
    - zlib-devel
    - pcre-devel
    - openssl-devel
```
install目录下操作:  

\# cd /ansible-test/nginx/roles/install  

\# cp /usr/local/src/nginx-1.8.1.tar.gz ./files/  
 
\# cp /usr/local/src/nginx.conf ./templates/  

\# cp /usr/local/src/nginx templates/  

\# vim vars/main.yml  

```
nginx_user: www
nginx_group: www
nginx_bin: /usr/sbin
nginx_ver: nginx-1.8.1
nginx_basedir: /etc/nginx
```

\# cd tasks  

\# vim copy.yml  

```
- name: Copy Nginx Software
  copy: src=\{\{ nginx_ver \}\}.tar.gz dest=/tmp/\{\{ nginx_ver \}\}.tar.gz owner=root group=root
- name: Uncompression Nginx Software
  unarchive: 
    src: /tmp/\{\{ nginx_ver \}\}.tar.gz
    dest: /tmp/
    remote_src: yes
- name: Setting Nginx Configure
  shell: ./configure --prefix=\{\{ nginx_basedir \}\} --sbin-path=/usr/sbin/nginx --conf-path=\{\{ nginx_basedir \}\}/nginx.conf --without-http_memcached_module --with-mail_ssl_module --with-http_flv_module --with-http_dav_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_gunzip_module --user=\{\{ nginx_user \}\} --group=\{\{ nginx_group \}\} --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-pcre --error-log-path=\{\{ nginx_basedir \}\}/logs/error.log --http-log-path=\{\{ nginx_basedir \}\}/logs/access.log
  args:
    chdir: /tmp/\{\{ nginx_ver \}\}
- name: Compile Nginx
  shell: make
  args:
    chdir: /tmp/\{\{ nginx_ver \}\}
- name: Install Nginx
  shell: make install
  args:
    chdir: /tmp/\{\{ nginx_ver \}\}
- name: Copy Nginx Start Script
  template: src=nginx dest=/etc/init.d/nginx owner=root group=root mode=0755
- name: Copy Nginx Config
  template: src=nginx.conf dest=\{\{ nginx_basedir \}\}/ owner=root group=root mode=0644
- name: Create Nginx config path
  file: 
    path: /\{\{ nginx_basedir \}\}/conf.d
    state: directory
    mode: 755
```

\# vim install.yml  
 
```
- name: Create Nginx user
  user: name=\{\{ nginx_user \}\} state=present createhome=no shell=/sbin/nologin
- name: Add Boot Start Nginx Service
  shell: chkconfig --level 345 nginx on
- name: Delete Nginx compression files
  file: 
    path: /tmp/\{\{ nginx_ver \}\}.tar.gz
    state: absent
- name: Delete Nginx Decompression directory
  file: 
    path: /tmp/\{\{ nginx_ver \}\}
    state: absent
```

main.yml 配置包含 copy.yml 和 install.yml  

\# vim main.yml  

```
- import_tasks: copy.yml
- import_tasks: install.yml
```
4、 编辑总入口文件  

\# cd /ansible-test/nginx  

\# vim install.yml  

```
---
- hosts: jekyll-blog
  remote_user: root
  gather_facts: True
  roles:
    - common
    - install
```
5、 安装
ansible-playbook install.yml

#### 一、Ruby的安装


