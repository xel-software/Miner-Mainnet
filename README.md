----
# 欢迎来到 XEL!


XEL是基于密码学和区块链技术的分布式超级计算机.

----
## 免责声明


XEL CORE / XELINE是在主网上运行的开源软件，它们在运行期间有可能会出现错误或故障，其中有些可能会产生严重的后果。我们不承担任何因使用本软件或任何衍生作品直接或间接而导致的任何损害的任何责任。在阅读此处的信息后, 用户请自行承担软件的使用风险.。

*此代码的傅播只是希望它有所帮助，我们不会承诺作出任何适用或暗示式的用途保证。有关更多详细信息，请参阅GNU通用公共许可证.*


这是个miner原型，用于解决XEL的工作包。 **miner还没有做其他的优化**，因为，其目的是为了不局限于任何硬件或操作系统，来展示EPL的所有功能以及Miner和XEL节点之间的工作流程，

目的是使其他开发人员为特定硬件/操作系统创建适合的版本（包括所有适用的优化）来改进矿工的性能和功能。


**GPU miner是纯试验性的。如果您选择使用它，请密切监控您的显卡，以确保它们不会过热.**

----
## 使用源码运行XEL Miner

Miner		v0.9.6
EPL 	v0.9.1

**miner的创建已经在Ubuntu 16.04上使用GCC以及在Windows 7/10上使用MinGW32（使用GCC）进行了测试.**

### 安装先决条件

##### Linux

```
apt-get update
apt-get install -y cmake libcurl4-openssl-dev libudev-dev screen libtool pkg-config libjansson-dev libssl-dev
```

##### macOS

你必须先安装homebrew（如果尚未安装）：


```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

然后继续安装:

```
brew install gmp make cmake openssl
ln -s /usr/local/opt/openssl/include/openssl /usr/local/include/openssl
```

### 创建

```
git clone --depth 1 https://github.com/xel-software/xel-miner
cd xel-miner
```

如果不使用OpenCL:

```
cmake .
make install
```

如果使用OpenCL :

```
cmake .  -USE_OPENCL
make install
```


### 运行

##### 使用CPU:

`./xel_miner -t <num_threads> -P <secret_phrase> -D`

##### 使用GPU:

`./xel_miner -t <num_threads> -P <secret_phrase> -D --opencl`

use `./xel_miner -h` to see a full list of options.



----
## 帮助改进

-我们喜欢更多的推送/ pull requests

-我们也喜欢更多的问题/issues(提出必解决;-) )

-无论如何，都欢迎您提出任何的想法/ideas

-欢迎解决和跟进其他人提出的问题/issuse

-欢迎查看现有的代码和推送/pull requeset

----
## 故障排除

  - UI 错误或者堆栈复写 
    -可以在github上提交
----
## 信任性

 -miner的core是基于cpuminer
 -感谢Evil-Knievel的不懈努力，才创造出了EPL语言和工作包逻辑。
 -xel项目的更多內容可以在这里找到：https://github.com/xel-software
