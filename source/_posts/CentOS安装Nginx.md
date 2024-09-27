---
title: CentOS安装Nginx
q: true
i: libffi-devel
date: 2024-09-27 17:31:30
tags:
    - CentOS
    - Nginx
---
1.添加EPEL仓库： CentOS默认的仓库中可能不包含Nginx，所以需要添加EPEL（Extra Packages for Enterprise Linux）仓库。
```shell
sudo yum install epel-release
```
2.安装Nginx： 使用yum命令安装Nginx。
```shell
sudo yum install nginx
```
3.启动Nginx服务： 安装完成后，启动Nginx服务。
```shell
sudo systemctl start nginx
```
4.设置Nginx开机自启： 如果希望Nginx随系统启动而自动运行，可以使用以下命令。
```shell
sudo systemctl enable nginx
```
5.验证Nginx安装： 检查Nginx服务的状态，以确保它正在运行。
```shell
sudo systemctl status nginx
```