#### 五、几个概念
类似于EOS，filecoin也有以下几个区块链的基本标准概念：
[![kHe279.md.png](https://s2.ax1x.com/2019/02/28/kHe279.md.png)](https://imgchr.com/i/kHe279)
##### 1、节点（node）
启动方式：
```
go-filecoin daemon
```

节点有唯一的ID，查看方式：
```
go-filecoin id
```

##### 2、帐户（account）
账户（account）是记账的基本单位，账户ID和节点ID一一对应，和钱包地址一对多关系（即一个节点账户下，可以有多个钱包地址），账户直接和交易FIL代币挂钩，由节点初始化时生成。

##### 3、钱包（wallet）
与以太坊、EOS一样，钱包是链上的一对公私钥对。钱包地址即是公钥，需要持有与之配对私钥的人才能操作该钱包内的资产。

通过下面命令，查询账号内所有钱包地址。
```
go-filecoin wallet addrs ls
``` 
或者 
```
vi ~/.filecoin/config.json
```

下面会详细介绍钱包操作。

##### 4、矿工（miner），在EOS里叫BP（区块生产者 block producer）
与账户和钱包不同，矿工（Miner）相关信息和配置并不会在部署Filecoin节点时自动创建，而是需要通过以下命令手工创建：
```
go-filecoin miner create 10 100 --price=0 --limit=1000 --peerid `go-filecoin id | jq -r '.ID'`
```
以上命令中，指令 `go-filecoin id | jq -r '.ID'` 会筛选出节点ID（即Qm开头的），这说明矿工始于节点绑定的，而不是帐号或者钱包。

#### 六、钱包操作
##### 1、创建新钱包
```
go-filecoin wallet addrs new
```
##### 2、查询现有钱包
```
go-filecoin wallet addrs ls
```
##### 3、导出节点钱包
```
go-filecoin wallet export <address>
```

##### 4、导入其他钱包
```
go-filecoin wallet import <wallet_file>
```

##### 5、获取测试代币（FIL Mock）
无论是比特币还是ETH、EOS，所有区块链项目在主网上线前，都会有测试网络，即使主网上线后，也会有开发者网络。在这些非主网区块链上，需要做开发或者测试，也需要用到代币，但这类代币没有资产价值，都可以免费获得。这种免费获得的方式就是水龙头（Faucet）。

在Filecoin区块链上，可以通过水龙头获得FIL Mock测试代币。请注意：FIL Mock没有投资价值，只是做测试和开发用。

对于FIL，其基本作用只有两个：
* 对于存储需求方：使用FIL来与矿工进行存储交易，购买存储空间和时间。
* 对于存储提供方（矿工）：使用FIL作为抵押品，参与挖矿。

获取方法：
如果是devnet-user类型，打开网站：http://user.kittyhawk.wtf:9797，直接填入钱包地址即可。

每隔24小时都能重新获取一次。

获取后，会得到一个CID值，用以下命令同步：
```
go-filecoin message wait ${CID}
```

##### 6、查看FIL余额
```
go-filecoin wallet balance ${WALLET_ADDR}
```

#### 七、挖矿操作
##### 1、创建矿工
当账户里有FIL可以用来抵押后，执行：
```
go-filecoin miner create 10 100 --price=0 --limit=1000 --peerid `go-filecoin id | jq -r '.ID'`
```
越10几分钟后，启动完成。系统会分配一个矿工ID，如下图：
*****

与此同时，本机硬盘中会虚拟出一个空间共Filecoin使用。

##### 2、启动挖矿
```
go-filecoin mining start
```

##### 3、结束挖矿
```
go-filecoin mining stop
```
##### 4、查询交易
在 http://user.kittyhawk.wtf:8000/actors 网站上，搜索wallet ID查看交易。

##### 5、查询矿工ID
```
go-filecoin config mining.minerAddress | tr -d \"
```

##### 6、查询矿工对应的钱包
```
go-filecoin miner owner 矿工ID
```

##### 7、查看矿工对应的节点ID（即Qm开头的地址）
```
go-filecoin address lookup 矿工ID
```

##### 8、挂单，即发送Ask订单
```
go-filecoin miner set-price --from=fcq95ej5ufl6pnhrrk238l6nd8sshfqxmqc3uss3p(矿机主Wallet Address) --miner=fcq84hjsj2qjpa644x2jnc7lhz2hrelz0xuqr22yc(本地minerID) --price=0 --limit=1000 0.01(FIL/byte/block) 28800(duration)
```
说明：
* duration：根据区块估计的保存时间，平均每个区块30秒，因此如果设置为120，则为1小时；2880为1天

##### 9、获取挂单列表
```
go-filecoin client list-asks --enc=json | grep <minerID>
```

#### 八、文件操作
##### 1、上传文件
```
go-filecoin client import hello.txt
```
返回文件Hash值。

注意：这个Hash值与通过`ipfs add 文件名`得到的Hash值一模一样。

##### 2、浏览文件内容
```
go-filecoin client cat 文件Hash值
```

##### 3、下单，即将文件存储到矿机上
```
go-filecoin client propose-storage-deal <minerID> <文件Hash值> <askID> <duration>
```


##### 4、查询订单
```
go-filecoin client query-storage-deal <DealID>
```

##### 5、检索数据
```
go-filecoin retrieval-client retrieve-piece <minerID> <文件Hash>
```

##### 6、组合操作
如：要将一个文件A保存到minerId的矿机上：

第一步：上传文件A到filecoin网络：
```
go-filecoin client import A
```

第二步：获取报价：
```
go-filecoin client list-asks --enc=json | grep minderId
```
获取`AskId`即返回的JSON中的ID。

第三步：将文件存储到矿机商
```
go-filecoin client propose-storage-deal minderId QmA askId <duration>
```

#### 参考文档：
* https://github.com/filecoin-project/go-filecoin/wiki/Mining-Filecoin
