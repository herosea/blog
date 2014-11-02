title: hexo进阶
date: 2014-11-02 09:47:47
tags:
---

# github page使用HTTPS

[参考链接](https://konklone.com/post/github-pages-now-supports-https-so-use-it)

去掉CNAME文件, theme/light/layout/_partial/head.ejs里面添加。
```javascript
var host = "YOURDOMAIN.github.io";
if ((host == window.location.host) && (window.location.protocol != "https:"))
    window.location.protocol = "https";
```
我是通过看生成的源代码，找到文件头的特点，通过如下命令搜索到相关文件的
```bash
$ find . -type f -exec grep -H 'html5shiv.googlecode.com' {} \;
```
