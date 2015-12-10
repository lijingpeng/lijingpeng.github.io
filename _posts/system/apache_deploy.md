date: 2014-11-20
title: Apache禁止IP直接访问并绑定域名
tags: Apache
category: system
---

```bash
<VirtualHost 10.10.20.1:80>
   ServerAdmin webmaster@userver
   DocumentRoot /var/www
   Servername 10.10.20.1
   <Location /> 
    Order Allow,Deny 
    Deny from all 
   </Location> 
</VirtualHost>

<VirtualHost 10.10.20.1:80>
   ServerAdmin webmaster@userver
   DocumentRoot /var/www
   Servername www.domain.org
</VirtualHost>
 
<VirtualHost 10.10.20.1:80>
   ServerAdmin webmaster@userver
   DocumentRoot /var/www
   Servername domain.org
</VirtualHost>
```


第一段配置说明我们禁止所有的直接IP访问
第二段配置说明我们允许通过www.domain.org来访问站点
第三段配置说明我们允许通过domain.org来访问站点