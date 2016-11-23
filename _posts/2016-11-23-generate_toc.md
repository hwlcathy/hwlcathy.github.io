---
layout: post
title: Markdown 使用toc工具生成文章目录
categories: [blog ]
tags: [Read, ]
description: toc生成工具
---

#  markdown语法中的toc

markdown的语法中有 ： '[toc]' 可以生成文章的目录 ，但是git中是不会支持的 ， 其实我们可以在文章前面增加导航，每个标题都是锚点，但是手动拼写非常麻烦，这里有工具可以自动生成  



## 使用方法

- 下载工具

```shell
wget https://raw.githubusercontent.com/ekalinin/github-markdown-toc/master/gh-md-toc
chmod a+X gh-md-toc
```



- 处理已经写好的md文件

```shell
cat tkmonitor.md  | toc -  #这里toc已经设置别名
   * [线上监控](#线上监控)
      * [监控系统构成方式](#监控系统构成方式)
      * [目前单品引擎已有的监控](#目前单品引擎已有的监控)
         * [离线部分](#离线部分)
            * [概述](#概述)
            * [一. checkOdpsData](#一-checkodpsdata)
               * [如何监控一份odps数据？](#如何监控一份odps数据)
            * [二. checkHdfsData](#二-checkhdfsdata)
               * [如何增加监控hdfs数据？](#如何增加监控hdfs数据)
         * [实时更新监控](#实时更新监控)
            * [概述](#概述-1)
            * [如何接入新的实时更新任务 ？](#如何接入新的实时更新任务-)
               * [trace日志格式](#trace日志格式)
               * [trace日志的查询方式](#trace日志的查询方式)
               * [接入一个新的实时更新模块](#接入一个新的实时更新模块)

```

- 将工具输出拷贝到md文件头部即可
- 后续考虑进一步优化 ， 直接成型 
- 目前实验Linux可行，mac不行



## 工具来源

https://github.com/ekalinin/github-markdown-toc
