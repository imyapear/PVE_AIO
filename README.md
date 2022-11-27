# 简介
本教程适用于两个或以上网卡服务器配置PVE服务器，并在服务器中配置Openwrt软路由器，KODI应用，以及基于LXC容器的Docker服务。
由于教程面向新手，为了确保成功率和容易理解，请看教程前注意以下两点：

1. 教程中其中软路由以主路由模式存在于整体网络当中，并不适用于旁路有模式；
2. 教程中KODI应用直接安装于宿主主机，并未使用显卡直通的虚拟机作为载体；

如果对以上两点在意的话，可以考虑不继续观看。

本教程请与视频配套教程一同观看，视频仅限于本人[B站](https://space.bilibili.com/28457/)或[Youtube](https://www.youtube.com/channel/UCUWUYyNh8KFS7E6-Q0ajBzQ)渠道播放，拒绝转载于任何电商直播间或介绍页面，发现盗用将发动网友投诉，敬请注意。

- 教程视频地址（上）：https://www.bilibili.com/video/BV1GY41177Es/
- 教程视频地址（中）：https://www.bilibili.com/video/BV1M34y1e7SF/
- 教程视频地址（下）：https://www.bilibili.com/video/bv1c34y1E7nn/
- 辅助网络原理教程地址：https://www.bilibili.com/video/BV1r64y1q74R

## 下载PVE的ISO镜像 & 写盘
- 工具下载地址（百度）（提取码：2887）：https://pan.baidu.com/s/1q7bmqASSctg7HO2UW_W1Eg
- 工具下载地址（天翼）（提取码：xea7）：https://cloud.189.cn/t/jEzayaeuEVjy
## 安装PVE系统
## 换国内源：

PVE换源
```
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bullseye.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bullseye.gpg
echo "#deb https://enterprise.proxmox.com/debian/pve bullseye pve-enterprise" > /etc/apt/sources.list.d/pve-enterprise.list
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bullseye pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```

Debian换源
```
mv /etc/apt/sources.list /etc/apt/sources.list.bk
nano /etc/apt/sources.list
```
Sources.list加入源
```
deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free
# deb-src http://mirrors.ustc.edu.cn/debian stable main contrib non-free
deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free
# deb-src http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free

# deb http://mirrors.ustc.edu.cn/debian stable-proposed-updates main contrib non-free
# deb-src http://mirrors.ustc.edu.cn/debian stable-proposed-updates main contrib non-free
```

更新&安装ethtool
```
apt update
apt upgrade -y
```

如本地容量较小，可通过挂载远程NAS的iSCSI，NFS，SMB共享存储空间。（后期将有视频补完小技巧）