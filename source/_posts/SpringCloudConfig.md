title: Spring Cloud 配置中心
tags:
  - spring cloud
  - ''
  - ''
categories:
  - 技术
author: 周雄海
date: 2018-04-23 21:51:00
---

主要作用：集中化管理配置文件，方便维护管理。 微服务环境下，应用特别多，随之而来的配置文件也特别多，而且会出现很多一样的配置，比如kafka的配置，注册中心的配置，还有其他的公共服务。 集中在一起管理可以方便复用，用最小的配置量完成需求，而且方便维护和统一升级。 

后端实现：主要是git作为后端存储实际配置，也可以用svn、mysql等

主要三个概念： application、label、profile。可以理解每一个配置由这三个元素定位。

* application：应用。例如用户应用、购物车应用，对应到一个实体微服务
* label：版本。可以简单等同git 的 branch、tag或者commit Id。
* profile：特性？。和spring boot中意思一样
* 一个spring cloud应用要获取自己配置文件，会指定一个application（自己名字），一个label（可以有备选，按顺序选第一个存在的），多个profile

### 特性

### 优先级

由高到低： 远程、环境变量、config目录、jar包中

label优先级：左高于右，例如 feature-a,master，如果feature-a分支存在，只使用feature-a。简单理解为取得即返回。

profile优先级：左低于右，例如dev, test    合并dev和test，如果冲突，用test的。简单理解为覆盖式合并。

### 应用公共配置

配置文件名字为 ${application}-${profile}.(properties|yml)，例如 user-center-staging.yml。 特别的 application-${profile}.(properties|yml)为所有应用共享的配置文件。



## 实战

### 本地开发配置问题

**避免被远程覆盖，尽量每个配置都有profile**：有时本地开发会和测试连相同的配置中心，为了避免互相影响，建议所有会对程序运行有影响的配置文件都有profile。比如测试统一都有 test 一套profile。本地开发时不指定profile就可以安全的用本地配置。

### 环境区分

使用label 或者 profile都可以区分环境。 我们在实际应用中，和kubernate结合，测试环境会有多套，这个时候用profile来区分环境可以实现最小的配置区分环境。

另外测试、预发和生产的区分，基于权限的考虑，使用了不同的git 仓库。

### bootstrap.yml使用

建议应用的bootstrap中只配置如何连接配置中心，其余配置（包括注册中心）都放配置中心中。我在纠结选择注册中心还是配置中心放bootstrap中纠结了一阵，最终选择了配置中心。原因：

1. 配置中心可以一个小组一个自己维护，注册中心通常会作为公司统一的基础设施
2. 本地启动一个配置中心时依然可以选择是否注册到统一或者独立的注册中心

### 变量不存在时

${A:${B}}  这个表达式如果B不存在而A存在时不会得到A的值，需要写成 ${A:${B:C}}