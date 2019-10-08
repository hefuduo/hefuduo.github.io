---
layout:     post
title:      "使用maven工具生成aar的pom文件"
subtitle:   ""
date:       2019-04-00 19:19:00
author:     "LeoHe"
header-img: "img/post-bg-2015.jpg"
tags:
    - maven
---



背景:

​	某些情况下需要使用第三方的sdk, 但是第三方sdk大部分是将jar/aar文件离线提供的, 不方便公司内部各个APP 集成, 需要将aar上传到公司的maven仓库中, 可以使用mvn命令手动生成aar/jar文件的xml格式的pom文件.

使用如下



在${user}/.m2/repository目录下执行如下命令



```shell
mvn install:install-file 
#parameters
-Dfile=<your aar/jar file path> 
-DgroupId=<groupId> # com.xxx.xxx
-DartifactId=<artifactId> #project id
-Dpackaging=<package format> # aar / jar 
-DgeneratePom=<false or true> #genegerate pom file or not
-Dversion=<x.y.z> #versioncode

```



活学活用, 现学现用, 后面记录.