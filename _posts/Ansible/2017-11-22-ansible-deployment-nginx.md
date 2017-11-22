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
说明如下：                
roles 目录下有两个角色，common为一些准备操作，install 为安装nginx 的操作。
每个角色下有几个目录：
handlers :当发生改变时要执行的操作，通常用在配置文件发生改变，重启服务
files :为安装时用到的一些文件
meta :为说明信息，说明角色依赖等信息
tasks :核心的配置文件
templates :通常存一些配置文件，启动脚本等模板文件
vars : 定义的变量

6、文件处理
common:
cd common;mkdir tasks;cd tasks
vim main.yml
- name: Install initialization require software
  yum: name={{ item }} state=installed
  with_items:
    - gcc
    - gcc-c++
    - zlib-devel
    - pcre-devel
    - openssl-devel
install:
cd ../../install; mkdir tasks vars files templates
cp /usr/local/nginx.tar.gz files/
cp /usr/local/nginx/conf/nginx.conf templates/
cp /etc/init.d/nginx templates/
vi vars/main.yml
nginx_user: www
nginx_port: 80
nginx_basedir: /usr/local/nginx

cd tasks
vim copy.yml
- name: Copy Nginx Software
  copy: src=nginx.tar.gz dest=/tmp/nginx.tar.gz owner=root group=root
- name: Uncompression Nginx Software
  shell: tar zxf /tmp/nginx.tar.gz -C /usr/local/
- name: Copy Nginx Start Script
  template: src=nginx dest=/etc/init.d/nginx owner=root group=root mode=0755
- name: Copy Nginx Config
  template: src=nginx.conf dest={{ nginx_basedir }}/conf/ owner=root group=root mode=0644

vim install.yml
- name: Create Nginx user
  user: name={{ nginx_user }} state=present createhome=no shell=/sbin/nologin
- name: Start Nginx Service
  service: name=nginx state=started
- name: Add Boot Start Nginx Service
  shell: chkconfig --level 345 nginx on
- name: Delete Nginx compression files
  shell: rm -rf /tmp/nginx.tar.gz

main 配置包含 copy 跟 install
vim main.yml
- include: copy.yml
- include: install.yml

7. 编辑总入口文件
cd ../../nginx_install
vim install.yml
---
- hosts:192.168.32.105
  remote_user: root
  gather_facts: True
  roles:
    - common
    - install

8.安装
ansible-playbook install.yml

