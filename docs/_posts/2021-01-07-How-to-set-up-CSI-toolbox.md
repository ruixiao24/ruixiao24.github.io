---
layout: post
title: "How to set up CSI Tool"
date: 2021-01-27 14:57:00 -0000
categories: WiFi
---

# How to Set Up CSI Tool

记录一下配置CSI Tool的过程

## 安装环境

- X201

- Intel 5300网卡

- Ubuntu 14.04 **（注意要安装14.04而不是14.04.6，因为Linux内核版本最高只支持4.2）**

## 安装步骤

1. 完成https://dhalperi.github.io/linux-80211n-csitool/installation.html官方指南的第1、2 步

2. 安装python numpy、matplotlib、opencv-python、scipy、sklearn、PIL库（下载所有想下的，后面完成安装后只能连接没有密码的网络）

3. 下载微云上两个deb文件，先安装libgcrypt再安装aircrack

4. 完成https://dhalperi.github.io/linux-80211n-csitool/installation.html官方指南的第3、4 步

5. 按linux-80211n-csitool-supplementary/injection下的readme文件安装injection模式，安装完成后按腾讯微云上的recordingCSI.txt文件设置发送端和接收端（接收端不需要执行第二行）。如果接受端能够接收到数据则安装成功。

6. 发送端后面三个参数第一个为发送的包的总数，最后一个为发包的延时单位为微秒（现在为每10ms发一个包）

   ```bash
   sudo ./random_packets 1000000000 100 1 10000
   ```

6. 将微云上netlink文件夹下linux-80211n-csitool-supplementary/netlink里没有的.c文件拷贝到linux-80211n-csitool-supplementary/netlink下，覆盖makefile文件，重新make。然后按recordingCSI.txt重新测试接收端，如果接收到数据则安装成功。

## 系统测试

下载微云上csi_project代码。

1）  采集数据

接收端使用sudo ../netlink/log_to_file > 文件名 命令采集数据

将文件拷贝至csi_project下的dataset文件夹下

2）  运行read_data.py文件可以将csi源数据解析成csi数据，运行train.py可以训练模型，运行rec.py可以进行测试（最好采集无人时的环境数据作为环境类）