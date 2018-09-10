---
title: 下一代WEB协议——IPFS(星际文件系统)
date: 2018-09-10 15:42:59
tags:
- ipfs
- web
- 文件系统

categories:
- 区块链
- IPFS
---


# 简介

1991年，最早版本的HTTP(超文本传输协议)协议发布, 正式当时这个不怎么起眼的网络协议吹响了互联网行业的号角，随后而来的二十多年，HTTP彻底成为了互联网领域网络协议的绝对霸主。 但是，正如圈内人所言，没有绝对安全的系统一样，也不存在绝对完美的协议。随着时代的发展，HTTP协议爆发出来各种问题，顶尖的工程师们对其缺点越来越不满，抱怨也日益增多。突出体现与以下几个方面

<!--more-->
## HTTP低效并且昂贵
HTTP请求是从单一的计算机上下载文件，其产生的带宽费用要比基于P2P的分布式网络高出60%, ipfs通过P2P网络协议，除了能够节省带宽，而且能对数据进行备份保存。
## 人类的历史数据不能被永久保存，会被删除
基于HTTP协议的web服务器，其平均的生命周期只有100天。我们之前访问的站点，一旦服务关闭，我们的数据将永久消失，试想一下，随着QQ宠物的关闭下线，我们再也找不回曾经那些美好的记忆了。
## 中心化的web服务让权利掌握在少数人手中
网络已经称为了人类历史上最重要的发明之一，并且深深的影响着我们的生活。但是，不管我们是在淘宝开店还是在微信发表文章，我们都并不自由，核心的权利永远掌握在巨头的手中。
## 网络应用太依赖骨干网
传统的应用保证高可用都是高度依赖骨干网络，IPFS支持创建具有多种弹性的网络，无论是否具有Internet骨干网络，都可以实现持久的高可用。


创新的火种永远不会被扑灭，正是基于以上的种种的抱怨与矛盾，IPFS诞生了。在2014年的5月，一位来自墨西哥的小伙子Juan Benet发明了IPFS。正如绝大多数天才一样，他也有着亮眼的履历与背景，毕业于举世闻名的斯坦福大学，在2015年参与了大名鼎鼎的Ycombinator计划，并成功创办了Protocol Lab实验室。

尽管如此，但以上种种都不是IPFS被世人所熟知的原因，尽管IPFS一开始的设计就号称是革命性的技术。 但在其诞生的前几年，并没有引起世人的关注。 相信对区块链有所关注的人们都知道，2017年在区块链领域是非同寻常的一年，比特币价格一度暴涨至2万美金，各种ICO项目层出不穷。 正是这一年，基于IPFS的区块链项目FileCoin诞生了。FileCoin在2017年8月创纪录地募集到了2.5亿美金，而这仅仅是在Token sale出售了10%的代币的情况下，这也意味着FoinCoin还未上线，市值就达到了惊人的25亿美金，虽然这一记录很快就被后来的EOS所超越，但其所取得的成绩也是有目共睹。可以说FoinCoin的成功极大地促进了IPFS技术的发展，越来越多的开发者加入进来贡献代码，其影响力正在以指数般的速度增长。

# 理论介绍
## IPFS是什么？
打开ipfs的[官网网站](https://ipfs.io/)印入眼帘的几个大字，**IPFS is the Distributed Web**, 是的，在创始人严重其目的就是来挑战HTTP的霸主地位，定义下一代的web服务标准。但，随着社区的不断进化与创新，IPFS能干的事情已经隐隐超出了HTTP协议的范畴，除了传输其在存储领域也正闪耀了耀眼的光芒。当然，谁也无法把我历史的进程，IPFS最终能否像HTTP一样大方异彩还值得我们观察。但我相信IPFS，也正在践行着作为一个信仰者的使命。

前面提到IPFS的由来，其为了解决HTTP的痛点而来，所以IPFS有以下优点
- 高效而且廉价
- 记录永久保存
- 去中心化，机会均等
- 不依赖与主干网络，网络弹性

# IPFS安装
## Go 安装
go get github.com/ipfs/go-ipfs

## 源码安装

- 下载源码: [源码](https://dist.ipfs.io/#go-ipfs)
- 解压
```
tar xvfz go-ipfs.tar.gz
```
- 安装
```
cd go-ipfs
./install.sh
```

## 测试安装
```
ipfs --help
```
在终端有如下输出
![](https://diycode.b0.upaiyun.com/photo/2018/ad694024eccf06883ac2938f7f4132e6.png)

## Ipfs 初始化
```
ipfs init
```
![](https://diycode.b0.upaiyun.com/photo/2018/780b431b1906b6d7733974015c464f02.png)

# Quick Start

既然知道了IPFS是革命性的技术，那么作为入门，我们首先来看看，如何使用ipfs，以及我们能用ipfs做什么？

## 添加文件到ipfs
```
  echo "hello world" >hello
  ipfs add hello
```

## 浏览文件
```
  ipfs cat <the-hash-you-got-here>
```

## 添加一个文件夹
```
  mkdir foo
  mkdir foo/bar
  echo "baz" > foo/baz
  echo "baz" > foo/bar/baz
  ipfs add -r foo
```

## 浏览
```
  ipfs ls <the-hash-here>
  ipfs ls <the-hash-here>/bar
  ipfs cat <the-hash-here>/baz
  ipfs cat <the-hash-here>/bar/baz
  ipfs cat <the-hash-here>/bar
  ipfs ls <the-hash-here>/baz
```

## 查看链接信息
```
  ipfs refs <the-hash-here>
  ipfs refs -r <the-hash-here>
  ipfs refs --help
```

## 获取文件
```
  ipfs get <the-hash-here> -o foo2
  diff foo foo2
```

## IPFS对象
```
  ipfs object get <the-hash-here>
  ipfs object get <the-hash-here>/foo2
  ipfs object --help
```

## Pin + GC
```
  ipfs pin add <the-hash-here>
  ipfs repo gc
  ipfs ls <the-hash-here>
  ipfs pin rm <the-hash-here>
  ipfs repo gc
```

## 启动服务
```
  ipfs daemon  (in another terminal)
  ipfs id
```

## 网络
```
  (must be online)
  ipfs swarm peers
  ipfs id
  ipfs cat <hash-of-remote-object>
```

## 文件挂载
```
  (warning: fuse is finicky!)
  ipfs mount
  cd /ipfs/<the-hash-here>
  ls
```

## 工具
```
  ipfs version
  ipfs update
  ipfs commands
  ipfs config --help
  open http://localhost:5001/webui
```

## 浏览器
### webui
- http://localhost:5001/webui

### video

- http://localhost:8080/ipfs/QmVc6zuAneKJzicnJpfrqCH9gSy6bz54JhcypfJYhGUFQu/play#/ipfs/QmTKZgRNwDNZwHtJSjCp6r5FYefzpULfy37JvMt9DwvXse

###images

- http://localhost:8080/ipfs/QmZpc3HvfjEXvLWGQPWbHk3AjD5j8NEN4gmFN8Jmrd5g83/cs

###markdown renderer app

- http://localhost:8080/ipfs/QmX7M9CiYXjVeFnkfVGf3y5ixTZ2ACeSGyL1vBJY1HvQPp/mdown

# 总结
在本节内容里面，我们介绍了IPFS以及IPFS的安装使用，在quick-start里面的命令建议大家全部敲一遍，熟悉下IPFS，当然，仅仅如此相信大家都不会满意，所以在下一节会用IPFS写一个个人博客系统。真正的体会以下IPFS的魅力，等大家对IPFS感兴趣之后，我们在深入，讲讲IPFS的原理以及源码。

# 参考
- https://ipfs.io/docs/



