---
title: buildpacks 下个时代的企业镜像打包工具
mathjax: false
date: 2020-07-03 12:10:28
tags:
- buildpacks
- docker
- CI/CD
author: alfred.cai
---
由于中文对buildpack的介绍不是很多，于是我就来献丑了。
<!-- more -->
## 发布代码的流程
1. 最先我们的发布流程是登陆虚拟机进入部署，不同的环境不能很好的隔离，而且开发者在本地部署的环境和运维人员在虚拟机上的环境可能也会有细微的差别，又需要一段时间的沟通。
2. 后来有了docker的发布，能在同一台机器上隔离多个运行环境。并且能保证开发和运行环境一致。运维终于可以把锅甩回给了开发自己。

## 那么buildpack是开解决什么问题的呢？
现在微服务流行后，一个网站后端可能有数十个微服务的应用在运行。应用基本上是在统一的框架下编写不同业务逻辑代码。此时，dockerfile这个文件就会显得比较的累赘。数十个服务的dockerfile几乎都是复制黏贴到项目中。dockerfile中大部分内容的负责人应该是框架编写者而不是写业务代码的CURD boys。并且后续由于框架的升级对镜像的更新也需要去改写数十个项目的dockerfile。实在太繁琐了。

buildpacks也就由此流行起来了。他抛弃了dockerfile里对环境的具体操作，而是改成了一种描述性的表达。而具体要的环境又交回给了运维。

## 如何使用 buildpacks

### 打包代码
```bash
pack build my-app
    --path ~/codes/my-app
    --run-image cloudfoundry/run:base-cnb
    --buildpack org.cloudfoundry.openjdk
    --buildpack org.cloudfoundry.buildsystem
    --buildpack org.cloudfoundry.jvmapplication
    --buildpack org.cloudfoundry.tomcat
    --buildpack org.cloudfoundry.springboot
    --buildpack org.cloudfoundry.distzip
    --buildpack org.cloudfoundry.springautoreconfiguration
```
- pack 命令行
- my-app 打包的镜像名
- path 代码目录
- run-image 项目运行时的基础镜像
- buildpack 一系列预先定义好的编译和运行程序


为了简化代码，buildpacks又定义了一个builder的概念。builder里包含了一系列 buildpack，build-image（编译镜像）和 run-image（运行镜像）。
```bash
pack build my-app --builder cloudfoundry/cnb:bionic
```

### 选择可用的builder
```bash
pack suggest-builders
```

### 查看 builder 所包含的buildpack 和其顺序
```bash
pack inspect-builder gcr.io/buildpacks/builder 
```

## buildpacks 具体做了什么事情
buildpack一共有五个生命周期阶段
- detect 发现项目语言，找到一系列的buildpack
- analysis 分析文件，重用之前image layer
- restore 设置缓存
- build 执行buildpack文件打包文件创建image
- export 导出到docker或者远端镜像
 
五个步骤就是buildpack的打包的全过程了。

detect会去查找项目例如package.json, pom.xml, 

build.gradle来判断项目是用什么编程语言来选择buildpack文件到layers目录下。

analysis,resore 用于缓存image layer，metadata等信息加速打包过程。

build 执行buildpack打包镜像。 

export 发布镜像。

## buildpacks 历史
buildpacks 最早是Heroku公司在2011推出的。Heroku公司也是现在支持编程语言最多的。2018，Pivotal和Heroku公司共同定义了buildpacks协议，推出了Cloud Native Buildpacks(CNB)。至此已经有很多公司推出了buildpacks builder镜像了。例如[google](), [paketo](https://paketo.io/), [Cloud Foundry](https://docs.cloudfoundry.org/buildpacks/)
```
Google:                gcr.io/buildpacks/builder                    Ubuntu 18 base image with buildpacks for .NET, Go, Java, Node.js, and Python
Heroku:                heroku/buildpacks:18                         heroku-18 base image with buildpacks for Ruby, Java, Node.js, Python, Golang, & PHP
Paketo Buildpacks:     gcr.io/paketo-buildpacks/builder:base        Ubuntu bionic base image with buildpacks for Java, NodeJS and Golang
Paketo Buildpacks:     gcr.io/paketo-buildpacks/builder:full-cf     cflinuxfs3 base image with buildpacks for Java, .NET, NodeJS, Golang, PHP, HTTPD and NGINX
Paketo Buildpacks:     gcr.io/paketo-buildpacks/builder:tiny        Tiny base image (bionic build image, distroless run image) with buildpacks for Golang
```

## buildpacks协议规范
https://github.com/buildpacks/spec/blob/main/buildpack.md