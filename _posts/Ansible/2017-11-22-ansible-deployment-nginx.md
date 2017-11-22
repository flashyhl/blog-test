---
layout: page
title: Ansible安装Nginx 
permalink: /ansible-install-nginx/
---
 
-------
创建Ansible的安装目录
mkdir /ansible-test/nginx  

cd /ansible-test/nginx  

创建配置目录
mkdir -p roles/{common,install}/{handlers,files,meta,tasks,templates,vars}  

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

6、文件处理
common目录下操作：
cd /ansible-test/nginx/roles/common/tasks
vim main.yml
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
cd /ansible-test/nginx/roles/install
cp /usr/local/src/nginx-1.8.1.tar.gz ./files/
cp /usr/local/src/nginx.conf ./templates/
cp /usr/local/src/nginx templates/
vim vars/main.yml
```js
nginx_user: www
nginx_group: www
nginx_bin: /usr/sbin
nginx_ver: nginx-1.8.1
nginx_basedir: /etc/nginx
```

cd tasks
vim copy.yml
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

vim install.yml
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

main.yml 配置包含 copy.yml 跟 install.yml
vim main.yml
```js
- import_tasks: copy.yml
- import_tasks: install.yml
```
7. 编辑总入口文件
cd /ansible-test/nginx
```
vim install.yml
---
- hosts: jekyll-blog
  remote_user: root
  gather_facts: True
  roles:
    - common
    - install
```
8.安装
ansible-playbook install.yml

