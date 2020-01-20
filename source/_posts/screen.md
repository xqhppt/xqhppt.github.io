---
title: screen
date: 2017-06-16 22:17:08
tags: linux
categories: linux
---
## 背景
运行长时间的任务，在此期间不能断开连接或关闭窗口，不然任务就会被杀掉。

## 概念
screen是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。Screen就是一个远程会话管理工具，只要Screen不被关关闭，里面的会话也会保持。

## 安装

```
CentOS
    yum install screen
	Ubuntu
	    apt-get install screen
		```

## 基本用法
* 新建会话     `screen -S name`

* 暂时离开     `ctrl a+d`

* 恢复会话     `screen -r`

* 关闭会话     `exit`

* 清除dead     `screen -wipe` 

