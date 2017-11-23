---
layout: page
title: 使用Fortune命令创建文章
permalink: /deployment-blogsite/
---
 
-------
当前博客站点源码目录：/usr/local/src/blogsite  
创建自动产生文章的脚本：  
```
#!/bin/bash

source_code_path=/usr/local/src/blogsite
curr_date=`date +%F-%H-%m`
curr_date_n=`date '+%F %T'`
echo "---" > $source_code_path/_posts/Article/$curr_date-fortune.md
echo "layout: page" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo "title: $curr_date_n" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo "permalink: /$curr_date-fortune/" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo "---" >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo -e "\`\`\`" >> $source_code_path/_posts/Article/$curr_date-fortune.md
fortune -e fortunes chinese tang300 song100 >> $source_code_path/_posts/Article/$curr_date-fortune.md
echo -e "\`\`\`" >> $source_code_path/_posts/Article/$curr_date-fortune.md
```





