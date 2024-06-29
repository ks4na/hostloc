# 410 随身 wifi 刷 debian

- [登录](#登录)
- [修改 hostname](#修改-hostname)
- [设置静态 ip](#设置静态-ip)
- [换源和安装基础软件](#换源和安装基础软件)
- [时区调整](#时区调整)
- [默认语言调整](#默认语言调整)
- [开机脚本 /etc/rc.local](#开机脚本-etcrclocal)
  - [切换为 host 模式](#切换为-host-模式)
  - [重启 ModemManager](#重启-modemmanager)
- [挂载新分区](#挂载新分区)
  - [涉及 docker 时的额外处理](#涉及-docker-时的额外处理)
- [安装软件](#安装软件)
- [debian 系统下重刷](#debian-系统下重刷)
  - [重刷 debian](#重刷-debian)
  - [重刷 android](#重刷-android)
- [参考](#参考)

## 登录

随身 wifi 刷机后插入电脑，电脑正确配置 `RNDIS` （相关步骤见参考教程）后会自动连接以太网，查看 ip 后将最后一位改为 `1` (默认是 `10.42.1.1` ) 连接。

```sh
# 使用 `root` 用户登录，密码为 1313144
ssh root@10.42.1.1
```

登录后切换 `root` 用户：

```sh
# 修改 root 用户密码
passwd
# 删除 user 用户
userdel -r user
```

## 修改 hostname

```sh
echo xxx > /etc/hostname
```

重启后确认生效。

## 设置静态 ip

> 详细信息参考玩客云笔记

首先要删除自带的 `wifi` ：

```sh
nmcli conn delete wifi
```

然后连接 wifi，并设置静态 ip：

```sh
wifiname=<wifi_name>

nmcli dev wifi connect $wifiname password <密码>

nmcli conn modify $wifiname ipv4.method manual ipv4.address 192.168.1.99/24 ipv4.dns 192.168.1.1 ipv4.gateway 192.168.1.1

nmcli conn up $wifiname
```

> 如果有拓展坞，还需要修改 host 模式后，设置有线网。

## 换源和安装基础软件

更换为清华源：

```sh
cat <<EOF > /etc/apt/sources.list
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
EOF
```

连接网络之后，先 `apt update`，然后安装以下软件：

- vim
- bash-completion
- unzip
- rsync

```sh
apt install vim bash-completion unzip -y
```

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

## 开机脚本 /etc/rc.local

### 切换为 host 模式

向 `/etc/rc.local` 文件中 `exit 0` 之前加上这句：

```sh
# host 模式
echo host > /sys/kernel/debug/usb/ci_hdrc.0/role
```

重启后确认已经改为 host 模式。

### 重启 ModemManager

默认情况下，即使基带没问题，启动之后 `mmcli -L` 还是提示 `No modems were found`，需要重启一下 `ModemManager`:

```sh
systemctl restart ModemManager
```

每次启动后都应该执行该命令，所以将以下命令写入 `/etc/rc.local`:

```sh
# 启动 30s 之后重启 ModemManager
sleep 30
systemctl restart ModemManager
```

## 挂载新分区

在 `/etc/fstab` 中添加即可，示例：

```
UUID=<uuid> <mount-point> ext4 defaults,nofail,errors=remount-ro 0 2
```

> 其中加上了 `nofail` 避免磁盘问题导致启动失败。

### 涉及 docker 时的额外处理

如果挂载的分区存放了 docker 相关的内容（如 `data-root`，或者存在 docker 容器依赖的路径），那么为了避免开机时 docker 服务的启动早于该分区的挂载而导致 docker 容器启动失败，需要调整 systemd 中 docker 开机自启相关配置，等待分区挂载完成后再启动：

```sh
EDITOR=vim systemctl edit docker.service
```

然后打开的编辑器中顶部找到如下注释区域，在其内部添加以下内容（其中的挂载路径根据实际情况更改为指定的路径，例如上面挂载文件 `/etc/fstab` 中指定的挂载点为 `<mount-point>`）：

```sh
### Anything between here and the comment below will become the new contents of the file

[Unit]
RequiresMountsFor=/mnt/foo /mnt/bar

### Lines below this comment will be discarded
```

保存并退出后，输入以下命令确认更改内容已经保存成功：

```sh
cat /etc/systemd/system/docker.service.d/override.conf
```

下次重启时， docker 就会等待指定的目录挂载完成后再启动了。

> 问：如何移除本次更改？
>
> 答：使用 `rm /etc/systemd/system/docker.service.d/override.conf` 即可。

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
