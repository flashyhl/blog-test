---
layout: page
title: Dev、Staging站点的创建以及部署发布
permalink: /deployment-blogsite/
---
 
-------
站点规划  
Dev环境：  
访问方式：http://118.31.20.178:7701  
所在路径(已创建)：/data/httpd/devblog  
源码路径(已拉取)：/deployment/source_code/dev_blogsite  
Staging环境：  
访问方式：http://118.31.20.178:7700  
所在路径(已创建)：/data/httpd/stageblog  
源码路径(已拉取)：/deployment/source_code/staging_blogsite  

一、Dev、Staging站点的Nginx配置：  
1、Dev站点的Nginx配置：  
\# vim  /etc/nginx/conf.d/devblog.conf  
```
server {
    listen       7701;
    server_name  118.31.20.178;
    root   /data/httpd/devblog;
    index index.html;

    location / {
       access_log off;
       log_not_found off;
    }

    error_page  404 400 500 502 503 504  /404.html;
    location = 404.html {
        root   /data/httpd/devblog;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|ico|swf)$ {
        expires      5d;
    }

    location ~ .*\.(js|css)?$ {
        expires      1h;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log    off;
    }
        
    access_log  /data/log/nginx/acclog/devblog-access.log main;
    error_log   /data/log/nginx/errlog/devblog-error.log warn;
    server_name_in_redirect  off;
}
``` 
2、Staging站点的Nginx配置：  
\# vim  /etc/nginx/conf.d/stageblog.conf   
``` 
server {
    listen       7700;
    server_name  118.31.20.178;
    root   /data/httpd/stageblog;
    index index.html;

    location / {
       access_log off;
       log_not_found off;
    }

    error_page  404 400 500 502 503 504  /404.html;
    location = 404.html {
        root   /data/httpd/stageblog;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|ico|swf)$ {
        expires      5d;
    }

    location ~ .*\.(js|css)?$ {
        expires      1h;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log    off;
    }
        
    access_log  /data/log/nginx/acclog/stageblog-access.log main;
    error_log   /data/log/nginx/errlog/stageblog-error.log warn;
    server_name_in_redirect  off;
}
```
创建的站点配置文件如下：  
```  
\# ll /etc/nginx/conf.d/
total 10
-rw-r--r-- 1 root root  724 Nov 23 16:26 devblog.conf
-rw-r--r-- 1 root root  732 Nov 23 16:25 stageblog.conf
```

二、创建Dev、Staging的发布脚本： 
1、创建Dev的发布脚本  
发布脚本目录：  
\# mkdir /deployment/deploy_script -p  
发布脚本日志目录：  
\# mkdir /deployment/logs -p  
发布脚本：  
\# vim /deployment/deploy_script/deploy_dev_blogsite.sh  
```
#!/bin/bash

blogsite_source_path=/deployment/source_code/blogsite
outlog=/deployment/logs/blogsite_dev.log
tmp_log=/tmp/blogsite.txt
blog_websit=/data/httpd/devblog
deplog_script_path=/deployment/deploy_script

cd $blogsite_source_path
echo -e "\n### Script exe at `date +%F/%T` by `who am i|awk '{print $1" "$2" "$5}'` ###\n" >>$outlog

read -p "【更新Blogsite】请输入更新的GIT版本号,如果没有输入或10秒内无动作都将更新到最新版本:" -t 10 VER
if [ "$VER" == "" ];then
   git pull -s recursive -X ours >$tmp_log
   old_ver=`cat $deplog_script_path/blog_ver`
   new_ver=`git log | head -1 | awk '{print $2}'`
   if [ "$old_ver" = "$new_ver" ];then
     echo "没有信息被提交！"
     exit 1
   else
     echo "$new_ver" > $deplog_script_path/blog_ver
   fi   
     
elif  echo $VER |egrep -q "^[0-9A-Za-z]+$" ;then
   git reset --hard $VER >$tmp_log 
else
   echo "输入内容不符合要求,程序退出."
   rm -rf $tmp_log
   exit 1
fi

if [ $? -eq 0 ];then
   cat $tmp_log |tee -a $outlog
   echo  -e "\e[32;1m OK\e[0m GIT update 【$(git log |head -1)】" |tee -a $outlog
else
   echo  -e "\e[31;5m Fail\e[0m GIT update" |tee -a $outlog
   rm -rf $tmp_log
   exit 1
fi
rm -rf $blogsite_source_path/_site/*

jekyll build
rsync -vzrtopg --delete-after $blogsite_source_path/_site/ $blog_websit &>/dev/null

chown -R www.www $blog_websit
if [ $? -eq 0 ]
 then
   echo  -e "\e[32;1m OK\e[0m Modify files Owner" |tee -a $outlog
 else
   echo  -e "\e[31;5m Fail\e[0m Modify files Owner" |tee -a $outlog
fi
rm -rf $tmp_log
```


现在调整的差不多了就可以把站点代码放到GitHub仓库里
1、在自己的GitHub账号里新建仓库  
2、克隆前面创建的仓库   
```
git clone https://github.com/flashyhl/blog-test  
```
3、把之前修改好的站点文件拷贝到当前的blog-test目录里面   
4、提交站点到GitHub仓库   
```
git add .
git commit -m "jekyll pages commit"
git push origin master
```
这样就可以把Jekyll创建的站点上传到GitHub仓库里面



