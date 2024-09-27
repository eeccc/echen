---
title: CentOS安装Python环境
date: 2024-09-11 19:03:08
tags:
  - CentOS
  - Python
  - OpenSSL
---


## 前言
本篇提供一个在Centos7上部署一个Python环境的教程。Python程序被广泛开发与应用，在云服务器部署显得尤为重要，但部署过程繁琐且多坑。


## 安装OpenSSL 1.1.x
构建 Python 3.11 需要openssl 1.1.1及以上更新版本。系统存储库中可用的版本是旧的。
下载OpenSSL 1.1.1源码
```shell
wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
```
解压源代码
```shell
tar -zxvf openssl-1.1.1.tar.gz
cd openssl-1.1.1
```
配置和编译
```shell
./config --prefix=/usr/local/openssl-1.1.1 --openssldir=/usr/local/openssl-1.1.1
make
make install
```
配置环境变量（可选）
```shell
export LD_LIBRARY_PATH=/usr/local/openssl-1.1.1/lib:$LD_LIBRARY_PATH
```
验证安装
```shell
/usr/local/openssl-1.1.1/bin/openssl version
```
**配置全局生效**
编辑环境变量配置文件
```shell
sudo vi /etc/profile
```
添加以下行
```vim
export PATH=/usr/local/openssl-1.1.1/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/openssl-1.1.1/lib:$LD_LIBRARY_PATH
```
使更改生效
```shell
source /etc/profile
```
可通过如下指令查看OpenSSL版本
```shell
openssl version
```


## 下载libffi-devel库
运行如下指令
```shell
sudo yum install libffi-devel
```
查看版本
```shell
rpm -qi libffi-devel
```

## 安装Python3.11
下载Python3.11的源代码
```shell
wget https://www.python.org/ftp/python/3.11.9/Python-3.11.9.tgz
```
_如果下载比较慢，也可以从官网下载后通过SFTP传到服务器上。_
提取存档
```shell
tar xvf Python-3.11.2.tgz
```
进入到解压目录
```shell
cd Python-3.11*/
```
配置构建
```shell
LDFLAGS="${LDFLAGS} -Wl,-rpath=/usr/local/openssl/lib" ./configure --with-openssl=/usr/local/openssl
make
```
安装编译
```shell
sudo make altinstall
```
查看Python版是否安装成功
```shell
python3.11 --version
```
验证OpenSSL是否正常
```shell
python3.11
import ssl
ssl.OPENSSL_VERSION
```
查看pip版本
```shell
pip3.11 --version
```


## 虚拟环境
如有需要，可以参考。
_我还是喜欢跑全局的(~~大不了人跑~~)_
```shell
python -m venv myenv
source myenv/bin/activate
```
查看虚拟环境地址
```shell
echo $VIRTUAL_ENV
```
退出虚拟环境
```shell
deactivate
```