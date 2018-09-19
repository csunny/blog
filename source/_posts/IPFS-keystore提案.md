---
title: IPFS keystore提案以及命令解析
date: 2018-09-19 15:10:15
tags:
- IPFS
categories:
- IPFS
---

## 目标
拥有一种安全，简单且用户友好的方式来存储和管理ipfs使用的密钥对。同时能够共享密钥、加密、解密、签名和数据认证。

<!--more-->
## 实施计划
### 存储
密钥将被存储路径在$IPFS_PATH下在名为keys的字典中。每一个密钥对都会存储在两个文件中，私钥文件$NAME和公钥文件$NAME.pub. 他们将被加密为PEM(或相似)的格式，并且可选密码加密。 一旦启动ipfs daemon， 密钥将按需延时加载。如果给的密钥是受密码保护的，在加载密钥时应提醒用户输入密码。$IPFS_PATH/keys文件夹因为是owner唯一可读，并且权限是700。密钥应该被owner唯一可读，并且权限是400。
### 接口
需要对IPFS工具链进行一些添加和修改以适应变更。首先创建两个ipfs子命令 ipfs key 和 ipfs crypt

```go
    ipfs key  - Interact with ipfs keypairs

SUBCOMMANDS:
    ipfs key gen     - Generates a new named ipfs keypair
    ipfs key list    - Lists out all local keypairs 
    ipfs key info <key>  - Get information about a given key
        ipfs key rename <key> <name>  - Rename a given key
        ipfs key show <key>    - Print out a given key
        ipfs rm   <key>     - Delete a given key from your keystore

        ipfs key send <key> <peer>  - Shares a specified private key with the given peer
    
    Use 'ipfs key <subcmd> --help for more information about each command.'

DESCRIPTION:
'ipfs key' is a command used to manage ipfs keypairs.

```

```
    ipfs crypt  - Perform crytographic operations using ipfs keypair.

SUBCOMMANDS:
    ipfs crypt sign <data>   - Generates a signature for the given data with a specified key 
    ipfs crypt verify <data> <sig>  - Verify that given data and signature match 
    ipfs crypt encrypt <data>   - Encrypt the given data
    ipfs crypt decrypt <data>   - Decrypt the given data

DESCRIPTION:
    `ipfs crypt` is a command used to perform various cryptographric operations using ipfs keypairs, including: signing, verifying, encrypting and decrypting.
```

### 一些子命令
#### Key Gen
```
    ipfs key gen    - Generate a new ipfs keypair

OPTIONS:
    -t, -type        string  - Specify the type and size of key to generate 
    -p, -passphrase string    - Passphrase for encrypting the private key on disk 
    -n, -name    string       - Specify a name for the key

DESCRIPTION 
    'ipfs key gen' is a command used to generate new keypairs.
    IF any options are not given. the command will go into interactive mode and prompt the user for the missing fields.
```

#### 注释
与ssh-keygen类似。 

#### key send
```

    ipfs key send <key> <peer> - Send a keypair to a given peer

OPTIONS:

	-y, -yes		bool		- Yes to the prompt

DESCRIPTION:

    'ipfs key send' is a command used to share keypairs with other trusted users.

	It will first look up the peer specified and print out their information and
	prompt the user "are you sure? [y/n]" before sending the keypair. The target
	peer must be online and dialable in order for the key to be sent.

	Note: while it is still managed through the keystore, ipfs will prevent you from
			sharing your nodes private key with anyone else.`
```
#### Crypt Encrypt
```
ipfs crypt encrypt <data> - Encrypt the given data with a specified key

ARGUMENTS:

	data						- The filename of the data to be encrypted ("-" for stdin)

OPTIONS:

	-k, -key		string		- The name of the key to use for encryption (default: localkey)
	-o, -output		string		- The name of the output file (default: stdout)
	-c, -cipher     string		- The cipher to use for the operation
	-m, -mode		string		- The block cipher mode to use for the operation

DESCRIPTION:

    'ipfs crypt encrypt' is a command used to encypt data so that only holders of a certain
	key can read it.
```

#### 其他的接口变更
我们还需要在其他命令中支持密钥，比如:
- ipfs add 
    - 支持--encrypt-key 操作, 用于块加密使用密钥添加的文件
        - 也可以添加 'encrypted' 节点在上面的跟unixfs节点
    - 支持 -sign-key操作, 从unixfs跟节点获得签名节点

- ipfs block put    
    - 支持 -encrypt-key 操作。 用于在散列和存储之间加密块。

- ipfs object put 
    - 支持 -encrypt-key 操作, 用于在散列和存储之间加密对象。 

- ipfs name publish 
    - 支持 -key操作，选择要发布到哪个密钥空间。

### 代码更改/添加

#### Repo
    - 添加keystore概念到repo, 安全的加载或存储密钥
    - 需要理解PEM编码

```go
type KeyStore interface {
    // Get a key from the cache
    GetKey(name string) (ci.PrivKey, error)

    // Save a new key into the cache, and write to disk
    StoreKey(name string, key ci.PrivKey) error

    // LoadKey reads the key from its file on disk, and stores it in the cache
    LoadKey(name string, password []byte) error
}
```

#### Unixfs
- 新节点类型 


### 结构
用于签名和加密的新DAG结构的一些临时模型(json表示)
签名的DAG
```python
{
    "Links": [
        {
            "Name": "@content",
            "Hash": "QMthecontenT",
        }
    ],
    "Data": protobuf{
        "Type": "Signed DAG",
        "Signature": "thesignature",
        "PubKeyID": "QimPubKeyHash",
    }
}
```

加密的DAG
```python
{
    "Links": [
        {
            "Name": "@content",
            "Hash": "QmRawEncryptedDag",
        }
    ],
    "Data": protobuf{
        "Type": "Encrypted Dag",
        "PubKeyID": "QmpubKeyHash",
        "Key": "emphemeral symmetric key, encrypted with public key."
    }
}
```


