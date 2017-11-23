---
layout: page
title: 创建Blog站点并且上传到GitHub仓库
permalink: /create-jekyll-blogsite/
---
 
-------
####一、创建jekyll站点：  
1、创建站点：  
\# mkdir /jekyll-blog   #创建目录
\# cd /jekyll-blog       
\# jekyll new blogsite  #建立博客站点 
``` 
Running bundle install in /jekyll-blog/blogsite... 
  Bundler: Don't run Bundler as root. Bundler can ask for sudo if it is needed, and
  Bundler: installing your bundle as root will break this application for all non-root
  Bundler: users on this machine.
  Bundler: The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x86-mswin32, x64-mingw32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x86-mswin32 x64-mingw32 java`.
  Bundler: Fetching gem metadata from https://rubygems.org/...........
  Bundler: Fetching gem metadata from https://rubygems.org/.
  Bundler: Resolving dependencies...
  Bundler: Using public_suffix 3.0.1
  Bundler: Using addressable 2.5.2
  Bundler: Using bundler 1.16.0
  Bundler: Using colorator 1.1.0
  Bundler: Using ffi 1.9.18
  Bundler: Using forwardable-extended 2.6.0
  Bundler: Using rb-fsevent 0.10.2
  Bundler: Using rb-inotify 0.9.10
  Bundler: Using sass-listen 4.0.0
  Bundler: Using sass 3.5.3
  Bundler: Using jekyll-sass-converter 1.5.0
  Bundler: Using listen 3.0.8
  Bundler: Using jekyll-watch 1.5.0
  Bundler: Using kramdown 1.15.0
  Bundler: Using liquid 4.0.0
  Bundler: Using mercenary 0.3.6
  Bundler: Using pathutil 0.16.0
  Bundler: Using rouge 2.2.1
  Bundler: Using safe_yaml 1.0.4
  Bundler: Using jekyll 3.6.2
  Bundler: Using jekyll-feed 0.9.2
  Bundler: Using minima 2.1.1
  Bundler: Bundle complete! 4 Gemfile dependencies, 22 gems now installed.
  Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
New jekyll site installed in /jekyll-blog/blogsite. 
```
创建的站点目录如下：  
```  
.
└── blogsite
    ├── 404.html
    ├── about.md
    ├── _config.yml
    ├── Gemfile
    ├── Gemfile.lock
    ├── index.md
    └── _posts
        └── 2017-11-23-welcome-to-jekyll.markdown

2 directories, 7 files
```
#### 二、站点上传到GitHub仓库
1、创建Nginx的安装目录  
\# mkdir -p /ansible-test/nginx  
\# cd /ansible-test/nginx  


```
nginx_user: www
nginx_group: www
nginx_bin: /usr/sbin
nginx_ver: nginx-1.8.1
nginx_basedir: /etc/nginx
```



