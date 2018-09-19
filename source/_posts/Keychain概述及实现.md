---
title: Keychain概述及实现
date: 2018-09-19 13:41:00
tags:
- IPFS
categories:
- IPFS
---

## 简介
此文档介绍keychain, 一个分布式的merkle-linked 数据结构，链接加密的密钥、身份、验签、证书、密文、证明和其他的对象。

keychain提供了一种普通的构造来管理分布式的密钥和证书。 它跟公钥的基础设施类似，但进一步将对象做了绑定。
<!--more-->

```go
// Identity  represents an entity that can prove possession of keys.
// It is meant to map to People. Groups, Processes, etc. It is 
// essentially a Prover
type Identity struct {
    Name string  // does not need to be unique.
}

// Key represents a cryptographic key
type key struct{
    Algorithm Link   // the algorithm used to generate the key
    Encoding Link   // the encoding used to store the key
    Bytes  Link     // the raw key bytes
}

// Key pair represents a pair of keys.
type Keypair struct {
    Public Link   // the public key
    Secret Link    // the secret key
}
// Signature represents a digital signature over another object
type Signature struct {
    Key Link // the key used to verify this signature(PublicjKey)
    Algorithm  Link // the algorithm used to sign the signee
    Encoding Link  // the encoding the sig is serialized with 
    Signee Link   // the object the key is signing
    Bytes Data  // the raw signature bytes
}
// Ciphertext represents encrypted data
type Encryption struct {
    Decryptor Link  // the identity able to decrypt the encrytion 
    Ciphertext Link  // the encrypted data
}
```

## proof 类型
```go
// proofOfControl proves a certain key is under control of a prover
var ProofOfControl = "proof-of-control"

// ProofOfWork proves an amount of work was expended by a prover.
var ProofOfWork = "proof-of-work"

// ProofOfStorage proves certain data is possessed by prover.
var ProofOfStorage = "proof-of-storage"

// ProofOfRetrievability proves certain data is possessed by _and retruevable from _ a prover.
var ProofOfRetruevability = "proof-of-retruevability"
```


## 图解
