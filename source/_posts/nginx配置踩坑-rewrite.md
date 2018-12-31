---
title: nginx配置踩坑--rewrite
date: 2018-09-05 18:52:43
tags:
---
问题
---
在协助部署吉林wap商城代码时，nginx配置出现一个问题；
通常前端框架的nginx配置如下：
```
        location ~* \.(html|htm)$ {
            root /emallapp/Mobile_mall;
            etag off;
         if ($request_uri ~* "^/([^/]*)/([^/]*).html") {
                set $appid $2;
                rewrite ^/([^/]*)/([^/]*).html /$2.html;
            }
            if ($request_uri ~* "^/([^/]*)/([^/]*)/([^/]*).html") {
                set $appid $3;
                rewrite ^/([^/]*)/([^/]*)/([^/]*).html /$2.html;
            }
            index index.html index.htm;
            expires 1s;
        }
```
因为要增加上下文，需要在匹配html前先把上下文处理掉，否则资源文件无法找到：
```
locatio~^/mall/ {
        rewrite ^/mall/(.*)$ /$1 last;
}
```
问题就出现在这里，增加了这段代码后，上面的html匹配不到了

原因
---
在nginx中，
* $request_uri是初始的uri，并且不随rewrite变化
* $uri  是随rewrite变化的uri
而rewrite中，匹配的链接就是$uri
在上面的配置中，if条件的写法是正确的，而if条件体中rewrite这里出现了问题；

在没有处理上下文时，rewrite的写法是正确的，这也是导致我先入为主，认为配置是正确的；
但是，由于进行了上下文处理，rewrite后，uri已经发生了变化；

此时下面的rewrite中的正则表达式是改变后的$uri
因此正确的写法是这样：
```
        location ~* \.(html|htm)$ {
            root /emallapp/Mobile_mall;
            etag off;
         if ($request_uri ~* "^/([^/]*)/([^/]*).html") {
                set $appid $2;
            }
            if ($request_uri ~* "^/([^/]*)/([^/]*)/([^/]*).html") {
                set $appid $3;
                rewrite ^/([^/]*)/([^/]*).html /$2.html;
            }
            index index.html index.htm;
            expires 1s;
        }
```
因为处理了上下文，所以rewrite时需要减少一个匹配组

总结
---
rewrite中的匹配组对应的是$uri，收到rewrite的影响