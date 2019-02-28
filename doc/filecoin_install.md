## Filecoin节点安装与配置

##### 1、初始化节点

目前有三个类型的测试网络，分别是：devnet-user, devnet-nightly, devnet-test。

参考：https://github.com/filecoin-project/go-filecoin/wiki/Devnets

要启动一个filecoin节点，键入 `go-filecoin init --help`，查看帮助。
如如果要启动一个普通user节点，则如下命令:
```
go-filecoin init --devnet-user --genesisfile=http://user.kittyhawk.wtf:8020/genesis.car
```
默认的区块链数据保存在：`~/.filecoin` 目录下。

初始化节点只要执行一次即可。

##### 2、启动节点
```
go-filecoin daemon
```
可以将此命令在后台运行。
正常的话会看到不断刷新的节点，如下图：
![image](http://note.youdao.com/yws/res/11612/7F7FC10A933A42978DAFF3914168DD3B)

如果看不到节点刷新，请用:
```
rm -rf ~/.filecoin
```
删除 `~/.filecoin`目录后，重新执行一遍上面的 `go-filecoin init ...` 命令，再次初始化。


##### 3、设置节点昵称
节点启动后，开启另一个终端，执行:
```
go-filecoin config heartbeat.nickname "guqianfeng"
```
通过命令：`go-filecoin config heartbeat.nickname` 可以查询当前节点的名字。

##### 4、发现自己的节点
在另一个终端中，执行：
```
go-filecoin config heartbeat.beatTarget "/dns4/stats-infra.kittyhawk.wtf/tcp/8080/ipfs/QmUWmZnpZb6xFryNDeNU7KcJ1Af5oHy7fB9npU67sseEjR"
```
用于链接到Filecoin的网络监控节点。

然后打开：https://stats.kittyhawk.wtf/

就能在节点所在位置找到昵称命名的节点，如下图：
![image](http://note.youdao.com/yws/res/11617/45AD54AC9DDD4B45ABEA82E458DB37FB)
