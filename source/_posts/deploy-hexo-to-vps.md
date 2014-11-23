title: 在VPS上部署hexo
date: 2014-11-23 21:51:55
tags:
- vps
- hexo
  
---

原文地址：http://blog.berry10086.com/Tech/deploy-hexo-to-vps/

搭建博客时看了很多教程，大部分都是讲怎样部署到Github Pages，但是Github Pages有时候访问有问题，就想直接部署在VPS上，经过半天的搜索和摸索，找个一个比较好的方法。

## 基本思路 

有两种方法

1. 在VPS上执行hexo server，再配置Nginx反向代理，让blog的域名指向http://localhost:4000。
2. 在本地生成静态文件，把静态文件部署到VPS上，用Nginx直接做Web服务。

比较两种方法，第一种配置简单，但是更新博客比较麻烦，而且我使用了hexo-admin插件，如果直接在VPS上执行hexo server，就会暴露admin页面，另外第一种方案，服务器需要再开一个node进程，比较浪费内存。综合以上因素，选择了第二种方案。

## Nginx配置

在Nginx中新建虚拟主机

```bash
# touch /etc/nginx/sites-available/blog
# vi /etc/nginx/sites-available/blog
```

可以使用下面的配置，其中/var/www/blog是存放博客静态文件的目录，后面会用到

```bash
server {                                                                               
    listen 192.184.88.212:80 ;                                                                                               
    root /var/www/blog;                                                        
    server_name blog.berry10086.tk;                                                 
    access_log  /var/log/nginx/blog_access.log;                                    
    error_log   /var/log/nginx/blog_error.log;                                            
    location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ {                            
            root /var/www/blog;                                    
            access_log   off;                 
            expires      1d;                            
        }                                                                              
    location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {                                   
        root /var/www/blog;                                                        
        access_log   off;                                                          
        expires      10m;                                                          
    }                                                                              
    location / {                                                                   
        root /var/www/blog;                                                
        if (-f $request_filename) {                                            
            rewrite ^/(.*)$  /$1 break;                                    
        }                                                                      
    }                                                                              
}
```
再执行
```bash
# ln /etc/nginx/sites-available/blog /etc/nginx/sites-enabled/blog
# /etc/init.d/nginx restart
```
这时Nginx已经可以提供服务了，如果通过scp或ftp把静态文件上传到/var/www/blog目录，博客就可以访问了。
## 自动部署
如果每次更新了博客，都需要通过scp或者ftp手动上传，想想就没有写博客的欲望了，翻阅hexo官方文档发现hexo支持rsync和git的部署方式。rsync不太了解，查了一下，配置起来有点麻烦，而且据说在windows下不好用，于是就选择了git，正好可以增进对于git的了解。
### Git安装
首先在VPS上安装git，`apt-get install git`

本地也要安装git，以windows为例，直接去官网下载即可

本地安装git后，有一个很重要的步骤，生成ssh密钥，按照下面的步骤进行

1. 打开`C:\Users\<用户名>\.ssh`文件夹，如果没有就新建
2. 在空白处单击右键，选择Git Bash Here打开终端
3. 输入命令`ssh-keygen -t rsa -C "blog"`一路回车，生成公钥和密钥，一会要用到公钥blog.pub

### 服务器端配置
1. 新建git用户,`# adduser git --ingroup sudo`
2. 切换到git用户，进行一些初始化操作
	```bash
	# su git
	$ cd ~
	$ mkdir .ssh && cd .ssh
	$ touch authorized_keys
	$ vi authorized_keys
	```
3. 把公钥blog.pub里的内容粘贴到authorized_keys里，这样本地和VPS之间就可以通过ssh使用git了
4. 测试，在终端中输入`ssh git@blog.berry10086.tk`，如果能够远程登陆，说明这一步没有问题
5. 为静态内容新建仓库，接着上面的步骤
```bash
$ cd ~ 
$ mkdir blog.git && cd blog.git
$ git init --bare
```
## 本地设置
设置git用户名，在Bash终端里
```bash
git config --global user.email "email@example.com"
git config --global user.name "username"
```
修改hexo配置文件里的deploy选项
```bash
deploy:
  type: git
  message: update
  repo:
    s1: git@blog.berry10086.tk:blog.git,master
 ```
运行hexo g hexo d，如果一切正常，静态文件已经被成功的push到了blog的仓库里，如果出现 * appears not to be a git repo* 的错误，删除hexo目录下的.deploy后再次`hexo g && hexo d`就可以了
Git hooks
本地deploy只是把静态文件push到了VPS的git仓库里。第一次搭建Git服务，以为push以后就可以在VPS的Git仓库里看到工作目录的内容，可是满心欢喜地cd ~/blog.git/branches后只看到空空如也的目录，心都凉了。push的文件去哪里了呢？
等等，既然blog.git是一个仓库，那么只要git clone /home/git/blog.git就可以取出仓库的内容了。顺着这个思路就有了下面的想法，使用git hooks在每次push完成后，执行一段脚本，把blog.git里的内容clone出来，再复制到/var/www/blog目录。
```bash
$ cd ~/blog.git/hooks
$ touch post-receive
$ vi post-receive
```
使用下面的脚本
```bash
#!/bin/bash -l
GIT_REPO=/home/git/blog.git
TMP_GIT_CLONE=/tmp/blog
PUBLIC_WWW=/var/www/blog
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
```
更改脚本权限和/var/www/blog权限
```bash
$ chmod +x post-receive
$ sudo chmod 775 -R /var/www/blog
```
这时在本地执行`hexo g && hexo d `几秒就可以完成博客的更新
## 参考内容
参考了以下的内容，感谢原作者
http://blog.rjjacky.info/2014/04/30/deploy-hexo-on-nginx/
http://ibruce.info/2013/11/22/hexo-your-blog/
