---
title: Ubuntu下编译安装Python
date: 2023-03-11 10:09:00
tags:
 - Ubuntu
 - Python
---

在Ubuntu上编译安装Python并不困难，最重要的一定要提前装好所有的依赖包，否则在用的时候就会遇到诸如ssl问题，找不到_bz2，lzma库等问题。

<!-- more -->

1. 安装依赖：

```bash
sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev libbz2-dev liblzma-dev
```

2. 下载源码(以Python3.7.9为例)并解压

```shell
wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz
tar -zxvf Python-3.7.9.tgz
```

3. 编译安装

```shell
# 进入目标文件夹
cd Python3.7.9
# 生成配置
./configure --enable-optimizations
# make
make
# 安装，强烈推荐使用altinstall而不是install，因为install会覆盖系统原有的python，这会影响系统的正常运行
sudo make altinstall
```
