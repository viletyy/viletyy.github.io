---
layout: post
title: "Golang：环境搭建"
date:  2019-12-20 10:28:00
author: "Viletyy"
header-style: text
tags:
  - Golang
  - Shell
---

### MacOS
去[下载地址](https://golang.org/dl/)下载，这里使用pkg包进行安装

在iterm中输入`go version`，能获取到go到版本号，则表示安装成功

### 配置环境变量
在iterm中查看是否存在 .bash_profile 或者 .zprofile
PS: 终端类型根据执行命令echo $SHELL显示的结果：

/bin/bash => bash => .bash_profile
/bin/zsh => zsh => .zprofile

编辑profile文件，添加以下代码
```shell
export GOROOT=/usr/local/go
export GOPATH=/Users/user/go_work
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GO111MODULE=on
export GOPROXY=https://goproxy.io,direct
```

输入`source ~/.bash_profile` 或者 `source ~/.zprofile` 使配置生效

输入`go env` 查看配置结果

### 一般项目启动命令
```shell
go mod tidy 
go mod vendor 
go run main.go
```