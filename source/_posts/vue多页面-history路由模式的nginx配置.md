---
title: vue多页面+history路由模式的nginx配置
date: 2018-11-06 18:17:56
tags:
---

背景
---
在拼团购业务中，遇到基于vue的多页面配置+history路由模式的场景，需要在nginx下进行配置


多页面配置
---
多页面配置是指，在spa项目中，有些特殊的页面需要独立处理，例如特殊的业务，或者页面的缩放程度不同；
配置多页面后，在前端项目中不再是只有1个html，那么需要在部署时，对nginx进行特别处理；


history模式
---
vue-router默认是hash模式，同时也提供了history模式，这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

当使用 history 模式时，URL 就像正常的 url，例如 http://yoursite.com/user/id

所以，要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面

nginx配置
---
如果多页面+history同时使用，该如何配置呢？

这是单页面的history模式的配置
```
location / {
  try_files $uri $uri/ /index.html;
}
```

多页面的话就需要属于另一个页面的路由必须可以通过url识别出来，例如单页面路由/a开头，那么多页面可以是/b开头；
但是由于我们场景，这个前端应用已经有一个统一的上下文/y，因此属于新页面的路由设计为/y/new

这样通过nginx就可以把多页面的路由区分开，进而加载不同的页面；

配置如下：
```
location /y {
  try_files $uri $uri/ /index.html;
}

location ^~/y/new {
  try_files $uri $uri/ /index2.html;
}

```
#### 注意
匹配新路由的location优先级要高于匹配原路由的location，因此加了^~