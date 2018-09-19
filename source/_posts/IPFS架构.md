---
title: IPFS架构概述
date: 2018-09-17 19:13:43
tags:
- IPFS
- 架构
categories:
- IPFS
---

## 目录

- IPFS and Merkle DAG
- Nodes and Network Model
- The Stack
- Applications and Datastructure   - on top of IPFS
- Lifetime of fetching an object
- IPFS User Interfaces
<!--more-->

## IPFS 和Merkle DAG

Ipfs的核心是MerkleDAG，是一个连接为hash值的有向无环图。这为IPFS中的所有对象都提供了如下特性:

- 认证   - 内容可以做hash运算并且可以通过连接hash值做反向验证。
- 持久   -一次拉取，永久缓存。
- 普适   - 任何数据结构都可以用merkledag来表达
- 去中心化 - 对象可以被任何人创建，而不需要集中的写入者。

反过来，这些特性使得系统具有如下优势：

- 连接是根据内容寻址的
- 对象可以由不受信任的服务提供
- 对象可以被永久缓存
- 对象可以被离线创建或使用
- 网络可以分区和合并
- 任何数据结构都可以建模并分发

IPFS是一个网络协议栈，用来组织代理网络来创建、发布、分发、服务和下载Merkledags，它是自证的、去中心化的、永久的web系统。

## 节点和网络模型

IPFS 网络使用基于PKI的身份认证。一个ipfs节点是一个程序包含查询、分发、和复制MerkleDag 对象。其标识由私钥定义。比如：

    privatekey, publicKey := keygen()
    nodeID := multihash(publicKey)

- MultiHash and upgradeable Hash

  在ipfs中所有的hash都被multihash编码，multihash是一个自描述的hash格式。 实际hash函数的使用，依赖与安全的需求。IPFS加密系统是可升级的，意味着hash函数出问题之后，网络可以迁移到更强壮的hash。天下没有免费的午餐，因为对象可能需要重新计算hash值，或者链接重新复制。 但是要确保不要假设原先预定义的hash长度可以适应未来的hash函数。

    sha2-256
    sha2-512
    sha3

## 技术栈

IPFS有一个模块化的技术协议栈。每一层都有多个实现。此规范仅仅定义层之间的接口，以及简要提及可能的实现。

IPFS有5层：

- 命令空间(naming)  - 自证的PKI命名空间（IPNS）
- MerkleDag  - 数据结构格式(thin waist)
- 交换  - 块交换和复制
- 路由   - 本地节点和对象
- 网络   - 在节点之间建立链接

![](stack-f1d6078b-9a67-4ae6-ae77-a71a6fbc6c0e.png)

这是一个从下到上的简易描述。

## 网络

网络提供点对点的传输在两个节点之间（可信或者不可信）。 主要处理：

- NAT遍历   - 打孔、端口映射和中继
- 支持多种传输协议  - TCP、 SCTP、UTP...
- 支持加密，签名或清除通信
- 支持加密、签名和清除通信
- 多路复用   - 连接复用、协议复用、peer复用

## 路由 — 发现节点和数据

IPFS路由层有两个重要的目的

- 节点路由     -找出其他节点
- 内容寻址     -查找分发到ipfs上的数据

路由系统是满足各种实现的接口，比如：

- DHTs
- mdns
- snr
- dns

## 块交换 — 传输内容寻址的数据

IPFS 块交换负责批量数据的传输。一旦节点之间相互连接并且认证之后，交换协议就开始传输数据。

块交换是满足以下实现的接口。比如：

- Bitswap   - IPFS中主要的交换协议。
- HTTP  - 一个简单的通过HTTP客户端和服务端实现的交易。

## MerkleDag

正如前面提到的，IPFS merkleDag 是IPFS核心的数据结构。它是边为hash值的有向无环图。它的另一个名称是Merkleweb。

Merkledag 数据结构如下所示：

    message MDagLink{
    	bytes Hash = 1;  //  multihash of the target object
    	string Name = 2;  // utf string name. should be unique per object
    	uint64 Tsize = 3 ; // cumulative size of target object
    }
    message MDagNode {
    	MDagLink  Links = 2;  //refs to other objects
    	bytes Data = 1;    //opaque user data
    }

Merkledag 是认证的“瘦腰”数据结构。它是最小的表示+传输任意认证数据结构的数据结构。更加复杂的数据结构都是利用merkledag来实现的。比如：

- git 以及其他的版本管理系统
- bitcoin 和其他的链
- unixfs 一个内容寻址的文件unix 文件系统

## MerkleDag 路径

merkledag足以解决路径问题

/ipfs/QmdpMvUptHuGysVn6mj69K53EhitFd2LzeHCmHrHasHjVX/test/foo

- (a) 首先会获取+解析QmdpMvUptHuGysVn6mj69K53EhitFd2LzeHCmHrHasHjVX
- (b) 然后查询(a)的链接，找到test的哈希，并解析
- (c) 然后查询(b)的链接，找到foo的哈希，并解析

## 命名  ——PKI命名空间和可变指针

IPFS主要关注内容寻址，这本质上是不可变的。改变一个文件也会改变它的hash值，也就改变了其地址，使它成为一个完全不同的对象。(可以把它想象成一个写时复制的文件系统)

IPFS命名层 — 或者IPNS

- 指向对象的可变指针
- 人类可读的名称

IPNS是基于SFS的。 它是一个PKI命名空间。名称只是公钥的hash值。掌控私钥的人掌控名称。记录被私钥签名并且分发(在IPFS中，通过路由系统)。这是在互联网上实现的一种基于平均主义的分配方式。即没有中心化的机构，也不存在证实颁发部门。

## 应用和数据结构——在IPFS的上层

到目前为止描述的技术栈足以用来表示任意的数据结构并在互联网上复制它们。同时也已经足以构建和部署去中心化的网站。

在IPFS上的应用和数据结构用merkledag来表示。用户可以创建任意的数据结构并且部署在全世界。

## Unixfs  —代表传统的文件

unix文件系统抽象 ——文件和目录——是人们在互联网上构建文件的主要方式。在IPFS中unixfs是表示IPFS之上的unix文件的数据结构。我们需要一个独立的数据结构来传递信息。

- 对象或者目录
- 总大小，减去索引的开销

## 获取对象的生命周期

假设我们IPFS中获取以下数据

    ipfs/QmdpMvUptHuGysVn6mj69K53EhitFd2LzeHCmHrHasHjVX/test/foo

IPFS 节点首先将路径拆分成如下形式：

    [ "QmdpMvUptHuGysVn6mj69K53EhitFd2LzeHCmHrHasHjVX", "test", "foo" ]

然后IPFS节点解析组件。第一个组件是/ipfs/... path总是multihash值，剩下的就是链接名称。

# IPFS用户接口

IPFS不仅仅是一个协议。它是一个工具集。IPFS实现包括处理Merkle的各种工具，如何部署、如何命名等等。这些接口可能对实现或整个项目的生存至关重要。这些接口控制人们如何使用IPFS。因此必须特别注意其设计和实现。比如：

- IPFS api   - 一个HTTP服务
- IPFS cli    - 一个unix 命令
- IPFS libs  - 各种语言的实现
- The IPFS gateways  - 互联网中通过IPFS提供的HTTP节点