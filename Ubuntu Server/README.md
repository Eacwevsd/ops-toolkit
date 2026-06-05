# Ubuntu Server 最小化安装与配置指南

本指南将使用 VMware 虚拟机，从零开始安装 Ubuntu Server 最小系统，并进行基础配置。

## 1. 系统镜像获取

首先进入 Ubuntu 官网：<https://ubuntu.com/download/server>

![Ubuntu 下载页面](./images/image-0.png)

点击 Download 下载完毕后即可得到如下的 ISO 镜像文件。

![ISO 镜像文件](./images/image-1.png)

## 2. 虚拟机准备 (以 VMware 为例)

首先单击“新建虚拟机”，选择“自定义”，随后点击“下一步”。

![新建虚拟机](./images/image-2.png)

弹出的页面继续点击“下一步”。

![下一步](./images/image-3.png)

选择“稍后安装操作系统”，点击“下一步”。

![稍后安装操作系统](./images/image-4.png)

选择“Linux”，版本选择“Ubuntu 64 位”，点击“下一步”。

![选择操作系统](./images/image-5.png)

自定义虚拟机的名称以及选择要存储系统的位置。

![命名虚拟机](./images/image-6.png)

根据自己电脑的实际配置情况，选择处理器数量和核数，这里以 2*3 为例，随后点击“下一步”。

![处理器配置](./images/image-7.png)

根据需要设置虚拟机的内存，这里以 4GB 为例，随后点击“下一步”。

![内存配置](./images/image-8.png)

网络连接类型选择“NAT 模式”，点击“下一步”。

![网络类型](./images/image-9.png)

默认选择（推荐选项，可能不一样！），点击“下一步”。

![I/O 控制器类型](./images/image-10.png)

默认选择磁盘类型（推荐选项，可能不一样！），点击“下一步”。

![磁盘类型](./images/image-11.png)

创建新的虚拟磁盘，选择“下一步”。

![选择磁盘](./images/image-12.png)

选择自己需要的磁盘大小（这里以 50GB 为例），随后点击“下一步”。

![磁盘大小](./images/image-13.png)

继续点击“下一步”。

![指定磁盘文件](./images/image-14.png)

单击“自定义硬件”，选择我们提前下载好的虚拟机镜像文件，点击“关闭”，准备安装。

![自定义硬件](./images/image-15.png)
![选择 ISO 镜像](./images/image-16.png)

## 3. 安装 Ubuntu 最小系统

开启虚拟机，在打开后的引导界面，选择第一项“Try or Install Ubuntu Server”。

![引导界面](./images/image-17.png)

弹出后的界面选择语言为 English。

![选择语言](./images/image-18.png)

出现下面的界面后，直接点击 Done。

![欢迎界面](./images/image-19.png)

使用上下键来选择安装的系统，这里选择最小安装（Minimized），当前面有×号后，点击 Done。

![选择最小安装](./images/image-20.png)

默认即可，我这里是 ens33，自动分配 IP 后点击“Done”。

![网络配置](./images/image-21.png)

Configure proxy 配置页面的 Proxy address 不需要配置，直接点击 Done。

![代理配置](./images/image-22.png)

这里选择阿里源，随后点击 Done。阿里云源地址：<https://mirrors.aliyun.com/ubuntu>

![选择镜像源](./images/image-23.png)

选择安装磁盘，直接回车默认自动分配，如果需要手动分区，选择 [custom storage layout]。

![磁盘分区](./images/image-24.png)

选中 free space → add gpt partition 创建分区。

![创建分区](./images/image-25.png)

创建 /boot 分区，2G，create 创建。

![创建 /boot 分区](./images/image-26.png)

同理创建 / 分区。

![创建 / 分区](./images/image-27.png)

检查磁盘分区后点击 Done。

![检查分区](./images/image-28.png)

弹出的窗口 Continue 继续。

![确认继续](./images/image-29.png)

设置用户名、密码以及计算机名，随后点击 Done。

![用户设置](./images/image-30.png)

是否要升级到 Pro 版本，这里选择暂时跳过（Skip for now）。

![Ubuntu Pro 选项](./images/image-31.png)

安装 OpenSSH 服务（用空格选中）。

![安装 SSH](./images/image-32.png)

预安装软件包，根据自己实际情况进行选择，如果没有，则 Done 继续。

![预安装软件包](./images/image-33.png)

开始安装。

![开始安装](./images/image-34.png)

稍等片刻，完成后点击 Reboot Now 进行重启。

![安装完成](./images/image-35.png)

**重要**：在虚拟机菜单栏选中虚拟机 → 右键设置 → CD/DVD 内，**取消勾选“启动时连接”**，确定。回到虚拟机窗口，按 Enter 后就会重启。

![取消连接 ISO](./images/image-36.png)
![重启界面](./images/image-37.png)

## 4. 登录系统

输入用户名和密码后登录。

![登录界面](./images/image-38.png)

登录后用 `ip a` 查看 IP。

![查看 IP](./images/image-39.png)

使用 shell 连接工具（如 MobaXterm）进行连接：会话 → SSH。

![MobaXterm 会话](./images/image-40.png)
![SSH 连接](./images/image-41.png)

## 5. 系统基础配置

### 5.1 设置 root 用户密码

使用命令：

```bash
sudo passwd root
```

输入当前用户的密码后，设置 root 的密码。

![SSH 连接成功](./images/image-42.png)

### 5.2 安装基础软件包

```bash
sudo apt install -y vim zip net-tools
sudo apt install iputils-ping
```

![安装完成](./images/image-43.png)

### 5.3 清理旧内核文件

```bash
sudo apt autoremove
```

![清理完成](./images/image-44.png)

### 5.4 配置静态 IP 地址

使用命令编辑 netplan 配置文件：

![编辑文件](./images/image-45.png)

```bash
sudo nano /etc/netplan/xxx.yaml
写入一下配置
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      addresses:
        - 192.168.93.135/24
      nameservers:
        addresses: [114.114.114.114, 8.8.8.8]
      routes:
        - to: default
          via: 192.168.93.2
  version: 2
```

保存后退出。

使用以下命令更新设置（由于 IP 变化，需要重新连接虚拟机）

```bash
sudo netplan apply
```

使用 Ping 命令测试网络连接（按 Ctrl + C 停止）：

```bash
ping www.baidu.com
```

![测试网络](./images/image-46.png)

### 5.5 更新软件包

```bash
sudo apt update
sudo apt dist-upgrade -y
```

![更新完成](./images/image-47.png)

### 5.6 时间同步

```bash
sudo timedatectl set-timezone Asia/Shanghai
date
```

## 最小系统安装完毕
