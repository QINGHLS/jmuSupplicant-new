# 小米路由器R3P-jmuSupplicant

原作者https://github.com/ShanQincheng/jmuSupplicant

[![License](https://camo.githubusercontent.com/232bcc81505d2ff5c9a69601c5dafc4df8d24e6857c4538b1772f77ce6c306ab/68747470733a2f2f696d672e736869656c64732e696f2f6372617465732f6c2f72757374632d73657269616c697a652e737667)](https://raw.githubusercontent.com/ShanQincheng/jmuSupplicant/master/LICENSE)

这是一个适用于集美大学的第三方锐捷认证客户端。关于实现此客户端的实现过程，可以参考[锐捷认证过程分析与第三方锐捷认证客户端的设计与实现](https://github.com/ShanQincheng/jmuSupplicant/blob/master/doc/锐捷认证过程分析与第三方锐捷认证客户端的设计与实现.pdf)

除了实现基础的认证并保持在线功能以外，额外实现了夜晚断网后认证功能。

普通认证支持所有服务类型的选择，夜晚断网后认证服务类型仅支持“教育网接入”。

经测试，12:00 p.m.后网速有较大提升，爱奇艺 1080P 勉强能够，抖音，微博毫无压力。

# 编译



## 普通编译



首先请确保系统已安装 `libpcap` 库以及 `CMake` 。（注ubuntu要安装libpcap-dev）

```
sudo apt update 
sudo apt install CMake libpcap-dev
```



```
git clone https://github.com/ShanQincheng/jmuSupplicant.git
cd jmuSupplicant
mkdir build
cd build
cmake ../
make
```



之后可以在 `build/bin` 目录下找到 jmuSupplicant 的可执行文件。

## 交叉编译



交叉编译需要先编译 libpcap ，之后再编译 jmuSupplicant。下面以交叉编译到 小米R3P路由器为例：(以下代码中的一些参数需要根据你的实际情况做相应的修改，仅供参考)

### 获取目标设备的交叉编译工具链



建议lede的源码编译

```
https://github.com/coolsnowwolf/lede
```

编译后可在/lede/staging_dir/中找到

### 配置环境变量



环境变量中的具体路径以及参数要根据你的实际情况做相应的修改，以下代码仅供参考：

我的目录是：/home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/  ，目录要正确

```
export PATH=$PATH:/home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/bin
export CC=mipsel-openwrt-linux-gcc
export CPP=mipsel-openwrt-linux-cpp
export GCC=mipsel-openwrt-linux-gcc
export CXX=mipsel-openwrt-linux-g++
export RANLIB=mipsel-openwrt-linux-ranlib
export LC_ALL=C
export LDFLAGS="-static"
export CFLAGS="-Os -s"
export STAGING_DIR=/home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl
export AR=mipsel-openwrt-linux-ar
```



### 交叉编译 libpcap



```
wget http://www.tcpdump.org/release/libpcap-1.9.0.tar.gz
tar zxvf libpcap-1.9.0.tar.gz
cd libpcap-1.9.0
./configure --host=mipsel-openwrt-linux --with-pcap=linux --prefix=$STAGING_DIR --disable-usb
make
```



如果交叉编译 libpcap 的过程中遇到错误，不用担心，这里我们只需要用到 `libpcap.a` ，编译后能得到该文件即可。之后将该文件以及 libpcap 的相关头文件复制到工具链的目录（要改成你自己的）中：

```
cp libpcap.a /home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/lib
cp pcap.h /home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/include
cp -r pcap /home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/include
```



### 交叉编译 jmuSupplicant

（需要 将目录替换成你自己的）

```
git clone https://github.com/ShanQincheng/jmuSupplicant.git
cd jmuSupplicant
mkdir build
cd build
cmake ../ -DCMAKE_FIND_ROOT_PATH=/home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/ -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=ONLY -DCMAKE_C_COMPILER=/home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/bin/mipsel-openwrt-linux-gcc -DCMAKE_CXX_COMPILER=/home/chen/Downloads/lede/staging_dir/toolchain-mipsel_24kc_gcc-8.4.0_musl/bin/mipsel-openwrt-linux-g++
make
```



之后可以在 `build/bin` 目录下找到 jmuSupplicant 的可执行文件。

# 使用说明



可以通过`--help`参数来获取程序运行帮助，下面举例两种使用情况：

## 正常使用



- 使用以下指令进行锐捷认证：

  ```
  sudo ./jmu -u学号 -p密码 -s0(教育网接入)1(联通宽带接入)2(移动宽带接入)3(电信宽带接入) -b
  ```

  

- 程序输出锐捷认证信息，或显示 login success， 则认证成功。

## 断网后的使用



- 首先自行找寻办公区域（夜晚能认证锐捷的地方，比如办公大楼）的 IP 地址，例如：123.123.123.123

- 使用以下指令进行断网后的锐捷认证：

  ```
  sudo ./jmuSupplicant -u学号 -p密码 -s0 -b -n --ip 123.123.123.123
  ```

  

- 程序输出锐捷认证信息，或显示 login success， 则认证成功。