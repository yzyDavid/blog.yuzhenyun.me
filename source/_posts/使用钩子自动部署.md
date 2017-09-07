---
title: 使用钩子自动部署
date: 2017-09-07 23:28:21
tags: github
---

如题，这篇post就是配置好自动部署以后的第一篇文章。现在我将博客的repo托管在github上，只要本地做好修改以后push到github，github的web hook请求我的服务器，一个clone-hexo generate-replace的脚本就会自动完成替换任务。

思路借鉴了这篇博客文章 http://www.cnblogs.com/lianer/p/5836194.html

这个脚本也放在github上面供大家参考了，链接为 https://github.com/yzyDavid/hexo-blog-deployer