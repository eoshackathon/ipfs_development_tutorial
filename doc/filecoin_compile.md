### *强烈建议使用海外服务器编译安装Filecoin*

#### 一、安装环境
##### 1、安装Go环境
因为编译`Filecoin`时，需要安装`golang 1.11.2`版以上环境，所以不建议使用
```
sudo apt install golang-go
```
命令，因为默认情况下，会安装`1.6.x`版，该版本不能正常运行`Filecoin`。

可以使用以下方式手动安装`golang 1.11.2`版：
1、去官网查看最新版链接 `https://studygolang.com/dl`
比如我要下的是 `https://studygolang.com/dl/golang/go1.11.5.linux-amd64.tar.gz`

运行命令:
```
wget https://studygolang.com/dl/golang/go1.11.5.linux-amd64.tar.gz
```

解压缩到`/usr/lib`:
```
tar -zxvf go1.11.5.linux-amd64.tar.gz -C /usr/lib
```

添加环境变量，输入命令：`vi /etc/profile`，如果你的go工作目录是`/root/go`，则在最后面添加如下配置:
```
export GOPATH=/root/go
export GOROOT=/usr/lib/go
export GOBIN=$GOPATH/bin
export GOTOOLS=$GOROOT/pkg/tool
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```
其中`/root/go`是工作目录，请事先创建该目录，并分别创建3个子目录：`src`, `pkg`, `bin`。

执行以下命令，使以上环境变量生效: 
```
source /etc/profile
```

最后执行:`go version`，如果返回：`go version go1.11 linux/amd64`，说明新的`go 1.11`版本已经安装成功。

##### 2、安装`rust`开发环境
由于`Filecoin`的复制证明工程实现，基本都是使用`rust`编写的，为此，我们需要安装`rust`环境。

执行：
```
curl https://sh.rustup.rs -sSf | sh
```

执行 `rustc --version` 和 `cargo --version` 查看版本，需要`1.32.0`以上的`rust`版本。

##### 3、安装`pkg-config`
执行：
```
sudo apt-get install pkg-config
```

##### 4、升级`gcc`版本到最新`7.4.0`版
执行：`gcc --version`查看版本，确保7.4.0以上版本，否则需要升级。
```
sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:jonathonf/gcc-7.4
sudo apt update
sudo apt install gcc-7 g++-7
```
安装完后，再执行：
```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 60 \
                         --slave /usr/bin/g++ g++ /usr/bin/g++-7 
sudo update-alternatives --config gcc
ls -la /usr/bin/ | grep -oP "[\S]*(gcc|g\+\+)(-[a-z]+)*[\s]" | xargs sudo bash -c 'for link in ${@:1}; do ln -s -f "/usr/bin/${link}-${0}" "/usr/bin/${link}"; done' 7
```
最后执行以下命令查看升级后的版本，是否为`gcc 7.4`：
```
gcc --version
g++ --version
```

##### 5、安装`gx`包管理工具
```
go get -u github.com/whyrusleeping/gx
```
注意，以上命令执行后，会在`${GOPATH}/bin/`目录中生成`gx`可执行文件，可以使用`gx --version`或者`./gx --version`命令查看gx的版本。

#### 二、用编译方式安装`go-filecoin`
1、创建go-filecoin项目目录，执行：
```
mkdir -p ${GOPATH}/src/github.com/filecoin-project
```

2、进入以上创建的目录中，克隆github上源码：
```
git clone https://github.com/filecoin-project/go-filecoin.git
```

3、运行以下命令，用来构建测试编译包：
```
go run ./build/*.go deps
```
备注：有时候`go`环境会配置交叉编译环境变量`GOARCH`和`GOOS`，这样会导致无法在`${GOPATH}/bin`目录下找到编译好的程序，从而出错。所以，需要临时将这两个环境变量设为空。如下：
```
env FILECOIN_USE_PRECOMPILED_RUST_PROOFS=true GOARCH= GOOS= go run ./build/*.go deps
```
下同。

4、第一次执行以上命令，会相当慢，约40分钟，但以后再次更新就会快了。

当看到以下红框，说明安装成功。
![image](http://note.youdao.com/yws/res/11568/B98DCC19DD42469986570F655227206C)

5、编译：
```
env GOARCH= GOOS= go run ./build/*.go go run ./build/*.go build
```
这个过程很快。

6、将程序安装到`${GOPATH}/bin`目录下：
```
env GOARCH= GOOS= go run ./build/*.go install
```

7、测试安装：
```
env GOARCH= GOOS= go run ./build/*.go test
```

8、验证安装是否成功，执行：
```
go-filecoin --help
```
如果出现帮助菜单说明，则说明安装成功。
