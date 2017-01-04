---
layout: post
title: 用tornado搭建http服务
categories: [blog ]
tags: [test, ]
description: 用tornado搭建http服务
---
# tornado搭建http服务

## 背景
需要搭建一个http服务来mock

要求：<br />

* 搭建过程简单
* 性能高（要做性能压测）


## tornado-helloWorld!

```
class HelloHandler(tornado.web.RequestHandler):
    def get(self):
        self.write('hello world !!')

class Hello2Handler(tornado.web.RequestHandler):
    def get(self):
        self.write("hello again !!")

application = tornado.web.Application([(r"/",HelloHandler),(r"/hello2",Hello2Handler)])
if __name__=="__main__":
    application.listen(4321)
    tornado.ioloop.IOLoop.instance().start()

```

Tornado是一个高效可扩展的非阻塞式web服务器以及其相关工具的开源版本，和当前主流的web服务器框架相比，明显的区别就在于它是非阻塞式服务器，而且速度相当快，这得益于它的非阻塞方式和对epoll的合理运用。简单的了解过后，我们来看下如何安装以及使用。<br />

