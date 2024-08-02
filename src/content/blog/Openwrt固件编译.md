---
title: OpenWRT 固件编译
description: OpenWRT 编译教程
pubDate: 02 08 2024
image: https://i.1m1ng.top/i/1/66aceda880a0e776fe863e3f079bd2fb6954f1f3330cc.webp
categories:
  - tech
tags:
  - OpenWRT
---

# 1 搭建编译环境

---

## 1.1 搭建前准备 (Linux)

建立登陆一个非Root但有sudo权限的账户

```
adduser openwrt
usermod -a -G sudo openwrt
```

接着登陆openwrt用户

```
su openwrt
```

---

## 1.2 安装依赖

安装编译环境相关依赖
```
sudo apt update
sudo apt upgrade
sudo apt-get install build-essential ccache ecj fastjar file g++ gawk gettext git java-propose-classpath libelf-dev libncurses5-dev libncursesw5-dev libssl-dev python python2.7-*dev* python3 unzip wget python3-distutils python3-setuptools python3-dev rsync subversion swig time xsltproc zlib1g-dev
```

---

# 2 feeds源

---

## 2.1 添加feeds源

在`lede`目录下的`feeds.conf.default`文件中添加以下feeds源

```
src-git kenzo https://github.com/kenzok8/openwrt-packages
src-git small https://github.com/kenzok8/small
```

---

## 2.2 更新和安装feeds源

```
./scripts/feeds update -a
```

```
./scripts/feeds install -a
```

通常更新和安装是一起执行的
```
./scripts/feeds update -a && ./scripts/feeds install -a
```

---

## 2.3 添加第三方feeds源

无luci界面的程序源码包，放进 `source/feeds/packages` 目录下

有luci界面的程序源码包，放进 `source/feeds/luci/applications` 目录内

然后运行下面的命令:

```
./scripts/feeds update luci
./scripts/feeds install -a -p luci
./scripts/feeds update packages
./scripts/feeds install -a -p packages
```

---

# 3 编译固件

---

## 3.1 编译配置

````
make menuconfig
````

---

## 3.2 下载DLL依赖库

```
make -j8 download V=s
```

---

## 3.3 编译

**WSL 需要提前执行以下环境变量**

```
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

开始编译

```
make -j$(($(nproc) + 1)) V=s
```

---

## 3.4 编译失败

编译失败就需要找到`Error`错误在哪里，一般都是插件上出问题

一般插件出问题就是没有Download下来或者是Download出错

反复重新下DLL依赖即可

还有一种方法能概率解决就是清理Tmp临时文件

```
rm -rf ./tmp
```

---

# 4 二次编译

---

## 4.1 更新Lead源码

```
git pull 
```

---

## 4.2 更新以及安装feed源

```
./scripts/feeds update -a && ./scripts/feeds install -a
```

---

## 4.3 重新配置

若要完全重新配置，需要先清理tmp文件夹和删除.config

```
rm -rf ./tmp && rm -rf .config
```

打开配置面板命令

```
make menuconfig
```

---

## 4.4 预下载DLL依赖

```
make -j8 download V=s
```

---

## 4.5 编译

```
make -j$(($(nproc) + 1)) V=s
```

---

# 5 额外配置

---

## 5.1 修改空间大小

```
Target Images ---> (16) Kernel partition size (in MB)    #内核部分占用
Target Images ---> (400) Root filesystem partition size (in MB)   #Root以及插件占用
```

一般内核部分给`64MB`以上，Root系统部分给`512MB`以上
