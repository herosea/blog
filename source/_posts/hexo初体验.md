title: hexo初体验
tags:
  - 学习
category: 工具
categories:
  - 工具
date: 2014-11-01 18:21:00
---
# 安装
参考官网文档： https://github.com/hexojs/hexo


## Installation

``` bash
$ sudo npm install hexo -g
```

## Quick Start

**Setup your blog**

``` bash
$ hexo init blog
$ cd blog
$ npm install
```

**Start the server**

``` bash
$ hexo server
```

**Create a new post**

``` bash
$ hexo new "Hello Hexo"
```

**Generate static files**

``` bash
$ hexo generate
```

## 配置
**.gitignore文件中添加**
```git
db.json                                                                                       
deploy
public
```
**_config.yml修改**
```yml
title: 享受生活 
author: 周雄海
email: herosea.zhou@gmail.com
language: zh-CN

url: http://herosea.me

date_format: YYYY-MMM-d
```

# 安装主题
在[主题列表](https://github.com/hexojs/hexo/wiki/Themes)中选择选择[light主题](https://github.com/hexojs/hexo-theme-light)
## 下载
Execute the following command and modify theme in _config.yml to light.
``` bash
$ git clone https://github.com/hexojs/hexo-theme-light.git themes/light
```
## 配置
**theme/light/_config.yml修改**
```yml
menu:
  主页: /
  档案: /archives
```
**_config.yml修改**
```yml
theme: light
```
# 发布
**添加CNAME**
新建source/CNAME文件，添加内容
```json
herosea.me
```
**_config.yml修改**
```yml
deploy:
  type: github
  repo: git@github.com:HeroSea/herosea.github.com.git
  branch: master
```
发布命令
```bash
$ hexo deploy
```


## More Information

- Read the [documentation](http://hexo.io/)
- Find solutions in [troubleshooting](http://hexo.io/docs/troubleshooting.html)
- Join discussion on [Google Group](https://groups.google.com/group/hexo)
- See the [plugin list](https://github.com/hexojs/hexo/wiki/Plugins) and the [theme list](https://github.com/hexojs/hexo/wiki/Themes) on wiki
- Follow [@hexojs](https://twitter.com/hexojs) for latest news

## License

MIT
