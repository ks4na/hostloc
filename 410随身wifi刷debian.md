# 410 随身 wifi 刷 debian

## 登录

随身 wifi 刷机后插入电脑，电脑正确配置 `RNDIS` （相关步骤见参考教程）后会自动连接以太网，查看 ip 后将最后一位改为 `1` (默认是 `10.42.0.1` ) 连接。

```sh
# 使用 `root` 用户登录，密码为 1313144
ssh root@10.42.0.1
```

登录后切换 `root` 用户：

```sh
# 修改 root 用户密码
passwd
# 删除 user 用户
userdel -r user
```

## 设置静态 ip

> 详细信息参考玩客云笔记

首先要删除自带的 `wifi` ：

```sh
nmcli conn delete wifi
```

然后连接 wifi，并设置静态 ip：

```sh
nmcli dev wifi connect <wifi名> password <密码>

nmcli conn modify <wifi名> ipv4.method manual ipv4.address 192.168.1.99/24 ipv4.dns 192.168.1.1 ipv4.gateway 192.168.1.1

nmcli conn up <wifi名>
```

> 如果有拓展坞，还需要修改 host 模式后，设置有线网。

## 安装基础软件

连接网络之后，先 `apt update`，然后安装以下软件：

- vim
- bash-completion
- unzip

## 时区调整

`timedatectl`

## 默认语言调整

```sh
# 添加需要的locale到locale.gen
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "zh_CN.UTF-8 UTF-8" >> /etc/locale.gen

# 生成locale
locale-gen

# 设置系统默认locale
update-locale LANG=en_US.UTF-8

# 重新配置locales包以应用更改
dpkg-reconfigure --frontend=noninteractive locales
```

重新登录后验证：

```sh
locale
```

## 开机切换为 host 模式

向 `/etc/rc.local` 文件中 `exit 0` 之前加上这句：

```sh
echo host > /sys/kernel/debug/usb/ci_hdrc.0/role
```

重启后确认已经改为 host 模式。

## 安装软件

- docker
  - apt 源方式安装，可以把命令中 `https://download.docker.com/linux/debian` 换成 `https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian` 后提高下载速度
  - 配置 `registry-mirrors`
  - 修改存储路径 `data-root` （`graph` 已废弃）

## debian 系统下重刷

### 重刷 debian

ssh 登录 debian 系统后运行以下命令进入 fastboot 模式：

```sh
reboot bootloader
```

后续可以执行刷机脚本刷入新的 debian 系统。

### 重刷 android

进入 9008 模式后，使用 miko 刷入备份的 android 系统镜像文件。

## 参考

- [酷安 - jsbsbxjxh66 - 410 随身 wifi 各个频率版&释放内存版&驱动全面](./assets/高通410刷debian/410随身wifi%20各个频率版&释放内存版&驱动全面%20来自%20jsbsbxjxh66%20-%20酷安.pdf)
- [随身 WiFi 概要及护理手册](https://post.smzdm.com/p/avxoegdm/)
- [HandsomeYingyan / OpenStick 项目](https://www.kancloud.cn/handsomehacker/openstick/2636505)
