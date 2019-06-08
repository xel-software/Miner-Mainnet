----
# 欢迎来到 XEL!


XEL是一种基于密码学和区块链技术的分散式超级计算机.

----
## 拒绝声明


XEL CORE / XELINE是在主网上运行的开源软件，但在它运行期间或者有可能出现错误或者故障，有些可能会有严重的后果。因此，我们不承担因任何直接或间接因使用本软件或任何衍生作品而导致的任何损害的任何责任。请自行承担使用软件和此处提供的信息。

*此代码的分发是希望它有用，但没有任何担保;甚至没有适销性或特定用途适用性的暗示保证。
有关更多详细信息，请参阅GNU通用公共许可证.*


这是解决XEL工作包的矿工原型。 **矿工没有以任何方式优化**，因为其目的是展示EPL的所有功能以及Miner和XEL节点之间的工作流程，无论硬件或操作系统如何。

其他开发人员希望通过为特定硬件/操作系统创建版本（包括所有适用的优化）来改进矿工的性能和功能。


**GPU矿工是高度实验性的。如果您选择使用它，请密切监控您的卡，以确保它们不会过热.**

----
## 从源代码运行XEL Miner

Miner		v0.9.6
EPL 	v0.9.1

**矿工构建已经在Ubuntu 16.04上使用GCC以及在Windows 7/10上使用MinGW32（使用GCC）进行了测试.**

### 安装先决条件

##### Linux

```
apt-get update
apt-get install -y cmake libcurl4-openssl-dev libudev-dev screen libtool pkg-config libjansson-dev libssl-dev
```

##### macOS

你必须先安装自制软件（如果尚未安装）：


```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

然后继续安装:

```
brew install gmp make cmake openssl
ln -s /usr/local/opt/openssl/include/openssl /usr/local/include/openssl
```

### 建立

```
git clone --depth 1 https://github.com/xel-software/xel-miner
cd xel-miner
```

如果你**不要**想使用OpenCL:
```
cmake .
make install
```

如果你想使用OpenCL :
```
cmake .  -USE_OPENCL
make install
```


### 跑

##### 使用CPU:

`./xel_miner -t <num_threads> -P <secret_phrase> -D`

##### 使用GPU:

`./xel_miner -t <num_threads> -P <secret_phrase> -D --opencl`

use `./xel_miner -h` to see a full list of options.



----
## 改进它

 我们喜欢**拉请求**
   - 我们喜欢问题（实际解决的问题;-)）
   - 无论如何，请确保你留下**你的想法**
   - 在问题跟踪器上协助其他人
   -  **审核**现有代码和拉取请求

----
## 故障排除

  - UI错误或Stacktraces？
  - 关于github的报道
----
## 积分

-该矿工的核心是基于cpuminer
-EPL / Work Package逻辑基于Evil-Knievel的不懈努力  
-xel项目可以在这里找到：https://github.com/xel-software
