
# Magsik v30 魔改版

主要去除 Magsik 特征（最大化）同时学习代码逻辑。


## 介绍

- **MagiskSU**: 替代传统 su 二进制，通过 Unix Socket 与 magiskd 通信处理权限请求
- **Magisk Modules**: Modify read-only partitions by installing modules
- **MagiskBoot**: The most complete tool for unpacking and repacking Android boot images
- **Zygisk**: 结合 LD_PRELOAD 和 PLT Hook 实现进程级注入


## 吐槽
官方编译教程就是一托狗屎，不接受反驳，最新版java21，文档写java17，没有详细过程，还有一堆问题。
```agsl
环境安装
1-0 sudo apt install -y git curl wget build-essential python3 python3-pip android-sdk-platform-tools zlib1g-dev libpcre3-dev zip unzip openjdk-17-jdk

1-1 Android SDK
wget https://dl.google.com/android/repository/commandlinetools-linux-10406996_latest.zip
unzip commandlinetools-linux-*.zip -d ~/android-sdk
mkdir -p ~/android-sdk/cmdline-tools/latest
mv ~/android-sdk/cmdline-tools/* ~/android-sdk/cmdline-tools/latest/
#配置环境变量
export ANDROID_HOME="$HOME/android-sdk"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools"
安装必备工具
sdkmanager --install "platform-tools" "build-tools;34.0.0" "platforms;android-34"




1-2 ondk
./build.py ndk
在执行命令的时候会告诉使用哪个版本的ndk：* Downloading and extracting ondk-r28.5-linux.tar.xz
如果失败：
cd $ANDROID_HOME/ndk/magisk
# 手动下载 ONDK（替换最新版本号）
wget https://github.com/topjohnwu/ondk/releases/download/r28.5/ondk-r28.5-linux.tar.xz
tar xvf ondk-r28.5-linux.tar.xz
mv ondk-r28.5/* .
rm -rf ondk-r28.5*


1-3 JDK-xxx
sudo apt install -y openjdk-xxx-jdk
卸载：`sudo apt remove --purge openjdk-17-jdk openjdk-17-jre`



1-4 rust安装
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustup --version
#使用Magisk 构建rust工具链
rustup toolchain link magisk "$ANDROID_HOME/ndk/magisk/toolchains/rust"
rustup default magisk

1-5 git配置
开启符号链接
git config --global core.symlinks true


2-1 git clone --recurse-submodules https://github.com/WMBa0/Magisk_Ba0.git

2-2
sudo gedit /etc/profile 
#添加Android SDK路径
export ANDROID_HOME=/home/wmba0/Android/Sdk
source /etc/profile

2-3
sudo ./build.py all


--------------------------------------问题1--------------------------------------
Magisk最新版(2025.7)需要glibc2.34的版本
报错：
/home/wmba0/Android/Sdk/ndk/magisk/toolchains/rust/bin/cargo: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.32' not found (required by /home/wmba0/Android/Sdk/ndk/magisk/toolchains/rust/bin/cargo)
/home/wmba0/Android/Sdk/ndk/magisk/toolchains/rust/bin/cargo: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.33' not found (required by /home/wmba0/Android/Sdk/ndk/magisk/toolchains/rust/bin/cargo)
/home/wmba0/Android/Sdk/ndk/magisk/toolchains/rust/bin/cargo: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by /home/wmba0/Android/Sdk/ndk/magisk/toolchains/rust/bin/cargo)
可以使用命令：apt-cache madison libc6  查看支持glibc最高的版本（我这里是最高支持2.31）
strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC_ 命令查看系统有没有glibc2.32-2.34
解决：
sudo gedit /etc/apt/sources.list
添加：deb http://mirrors.aliyun.com/ubuntu/ jammy main
sudo apt update
sudo apt install libc6


--------------------------------------问题2--------------------------------------
Magisk安装过程全程有权限问题，不知道什么情况，得用root
sudo -i
export ANDROID_HOME=/home/wmba0/Android/Sdk
./build.py all


--------------------------------------问题3--------------------------------------
* What went wrong:
Execution failed for task ':shared:compileDebugJavaWithJavac'.
> Java compilation initialization error
    错误: 无效的源发行版：21

这是java21

```