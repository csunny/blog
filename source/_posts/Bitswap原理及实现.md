---
title: Bitswap原理及实现
date: 2018-09-18 18:46:52
tags:

- IPFS
categories:
- IPFS
---

## 简介

Bitswap是IPFS的数据交易模型。它的目的是用来从其他节点请求或发送数据。Bitswap有两个主要的功能

1. 尝试从网络中获取已被客户端请求的数据。
2. 发送自己所拥有的数据给需要的节点。

## 介绍

Bitswap是IPFS的中心块交换协议。它处理IPFS用户、普通用户以及应用的请求，从网络中获取数据。它用来跟其他节点的Bitswap相互通信来交换彼此所需的数据。
<!--more-->
Bitswap是基于消息的协议，而不是回复。所有的消息包含需求清单、数据块。在接收到需求清单之后，IPFS节点或发出其拥有的数据给需求方。当接收到数据块之后，节点应该发送一个Cancel通知表明它不再需求相应的数据块。在协议层Bitswap是相当简单。

尽管Bitswap是一个相当简单的协议，考虑到时间和内存性能，也有很多值得关注的点。我们的目的是在这里尽量详细的记录这些，为了将来实现的时候可以不断迭代。

## 子系统

在Bitswap管理中两个主要的流程： 从其他节点请求数据和发送数据给其他节点。 数据块请求主要由需求管理员来调节，它告诉节点是否我们需要新的数据块。发送数据块主要由决策引擎处理，它决定了如何在节点之间进行资源分配。

![](Bitswap-18779f16-21e3-4d2b-8b69-8887a379dfe0.png)

## 类型

如下描述的类型是Bitswap子系统中描述的。

- CID: 使用特定数据块的内容寻址标识符
- Peer: 链接的另外的Bitswap实例
- Block： 二进制blob
- Message： Bitswap 消息
- Entry：在需求列表中，添加/删除特定的CID时可能包含在消息中的需求列表，其中包括
    - CID    一个特定的块
    - Priority    用户想要的CID的优先级
    - Cancel    一个bool值，旨在从我们的需求列表中删除CID。

Ledger：两个节点之间数据交换的聚合值。每个节点为其他节点存储一个Ledger。

## Bitswap消息

单个Bitswap消息可能包含以下任意内容

- 发件人的需求清单。此需求列表可能是发件人完整的需求列表，也可能仅仅是接收者需要知道的需求列表。
- 数据块。这些是接收者请求的数据块。

消息格式

    message Message{
    	message Wantlist {
    		message Entry{
    			optional string block = 1; // the block key 
    			optional int32 priority = 2; // the priority (normalized). default to 1
    			optional bool cancel = 3;   // whether this revokes an entry
    		}
    		repeated Entry entries = 1;  // a list of wantlist entries.
    		optional bool full = 2;   // whether this is the full wantlist. default to false.
    	}
    	optional Wantlist wanlist = 1;
    	repeated bytes blocks = 2;  
    }

## 需求管理者

需求管理者处理块请求。对于请求的块，通过cid认证，Bitswap.GetBlock(cid)方法被调用。Bitswap.GetBlock(cid)从网络中请求cid，如果收到相应的块，则返回。更具体来讲，Bitswap.GetBlock(cid)添加cid到需求列表中。需求管理者通过向每个节点的消息队列中添加条目来更新所有节点。

## 决策引擎

决策引擎如何在节点之间分配资源。当接收到一个需求列表时，消息会发送给决策引擎。对每一个需求列表中的CID，都有相应的块，块有一个任务添加到任务队列。一旦块被发送到节点的消息队列中，我们认为任务已经完成了。

在决策引擎中主要的数据结构是节点请求队列(PRQ)。PRQ添加节点到加权队列中，其中权重基于对一个或者多个节点的度量。这也是Bitswap策略的来源。当前，策略是一个函数，它的输入是ledger，输出是节点的权重。然后节点在各自的任务队列中提供任务。在给定的循环轮次中，每个节点的数据量由它们在队列中的秀昂队权重确定。

## 消息队列

每一个激活的节点都有一个相应的消息队列。消息队列保存要发送给节点的下一条消息。消息队列从其他的子系统中接收更新。

- 需求管理者：当CID从需求列表添加或删除时，我们必须更新节点。这些需求列表的更新被发送到其他所有节点的消息队列中。
- 决策引擎：当我们拥有节点需要的块，并且决策引擎决定发送这个块，我们广播这个块到其他节点的消息队列。

任务工作者观察消息队列，将消息从队列中出列，并且发送给相应的接收者。

## 网络

网络是表示通过一个或多个链接到我们的所有Bitswap节点的抽象。Bitswap消息流流入或流出的网络。在一个任意的网络中，我们必须假设我们所有的同伴都是理性的和自私的。

# 实现细节

## 消息合并

当已包含Bitswap消息的消息队列收到另一个消息队列时，新的消息应该与原消息合并，以减少发送单独数据包的开销。

## Bitswap会话

Bitswap会话是尝试优化发送到其他Bitswap客户端的块请求。当从网络中请求块图时，我们发送一个需求清单去更新包含根块给所有的节点。对于每个发送根块的节点，我们将此节点添加到图的激活集中。然后我们将图中的其他节点的所有请求发送到活动集中的节点。拥有图根节点的节点可能也会拥有其子节点，同时没有根节点的节点可能没有子节点。

实施建议

- 维护一组“在线合作”节点
- 协议侦听器接受伙伴接受消息流
- 协议发送方向合作伙伴打开流发送消息
- 分离出一个决策引擎，选择将哪些块、在何时、发送给哪些节点。

Sender

- 打开bitswap流
- 发送一个或多个bitswap消息
- 关闭bitswap流

Listener

- 接受bitswap流
- 接受一个或多个bitswap流
- 关闭bitswap流

Events

- bitswap.addedBlock(block)
    - 观察是否有任何节点需要此块，并发送
- bitswap.getBlock(key, cb)
    - 添加到需求列表
    - 可能会发送需求列表更新到其他节点
- bitswap.cancelGet(key)
    - 需求列表取消
- bitswap.receivedMessage(msg)
    - 处理需求列表变更
    - 处理块
- bitswap.peerConnected(peer)
    - 添加节点
- bitswap.peerDisconnected(peer)
    - 移除节点

Tricky Bits

- 客户端bitswap可能会调用getBlock 然后cancelBlock
- 合作伙伴可能会发送垃圾邮件
- 针对每个用户的标准化优先级

Modules

- bitswap-decision-engine
- bitswap-message
- bitswap-net
- bitswap-wantlist

Notes

    var bs = new BlockService(repo, bitswap)
    bs.getBlock(multihash, (err, block)=>{
    	// try to fetch from repo
    	// if not -> ask bitswap
    		// bitswap will cb() once the block is back, once. 
    			// bitswap will write to the repo as well.
    })

# 实现

- Go语言[实现](https://github.com/ipfs/go-ipfs/tree/master/exchange/bitswap)
- js[实现](https://github.com/ipfs/js-ipfs-bitswap)
