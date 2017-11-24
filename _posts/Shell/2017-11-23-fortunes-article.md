---
layout: page
title: 使用Fortune命令创建文章
permalink: /fortune-blogsite/
---
 
-------
当前博客站点源码目录：/usr/local/src/blogsite  
为了git能够自动push文章到GitHub仓库里，需要配置使用ssh方式拉取和推送。  
创建自动产生文章的脚本：  
vim /usr/local/src/site.sh  
  
```
#!/bin/bash

export LC_ALL=zh_CN.UTF-8
export LANG=zh_CN.UTF-8

source_code_path=/usr/local/src/blogsite
curr_date=`date +%F-%H-%M`
curr_date_n=`date '+%F %T'`
echo "---" > $source_code_path/_posts/Article/$curr_date-fortune.md
echo "layout: page" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo "title: $curr_date_n 创建" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo "permalink: /$curr_date-fortune/" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo "---" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo -e "\`\`\`" >> $source_code_path/_posts/Article/$curr_date-fortune.md
fortune -e fortunes chinese tang300 song100 >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo -e "\`\`\`" >> $source_code_path/_posts/Article/$curr_date-fortune.md

cd $source_code_path
git add .
git commit -m "提交$curr_date-fortune"
git push origin master
```
添加定时任务，每13分钟执行一次，如下：  
\# crontab -e  
```
*/13 * * * * bash /usr/local/src/site.sh
```
这样就能自动生成文章了





