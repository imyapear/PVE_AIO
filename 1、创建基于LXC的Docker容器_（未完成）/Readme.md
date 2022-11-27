# 创建CT：
## 1、CT模板换源
```
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.tuna.tsinghua.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```
重启服务
```
systemctl restart pvedaemon.service
```
## 2、下载模板、创建******特权********LXC

## 3、为容器加入渲染器硬件，并关闭AppArmor（部分显卡可能需要更新内核才能找到渲染器）
```
nano /etc/pve/lxc/[CT_ID].conf
```

加入硬件参数：（可先用ls -l /dev/dri查询）
```
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.apparmor.profile: unconfined
```

## 4、换Debian源
```
mv /etc/apt/sources.list /etc/apt/sources.list.bk
nano /etc/apt/sources.list
```

黏贴：
```
deb https://mirrors.ustc.edu.cn/debian/ bullseye main non-free contrib
deb-src https://mirrors.ustc.edu.cn/debian/ bullseye main non-free contrib
deb https://mirrors.ustc.edu.cn/debian-security/ bullseye-security main
deb-src https://mirrors.ustc.edu.cn/debian-security/ bullseye-security main
deb https://mirrors.ustc.edu.cn/debian/ bullseye-updates main non-free contrib
deb-src https://mirrors.ustc.edu.cn/debian/ bullseye-updates main non-free contrib
deb https://mirrors.ustc.edu.cn/debian/ bullseye-backports main non-free contrib
deb-src https://mirrors.ustc.edu.cn/debian/ bullseye-backports main non-free contrib
```

更新
```
apt update
apt upgrade -y
```

## 5、挂载远程smb
安装SMB组件并创建共享目录（nas_share可自定义）
```
apt install cifs-utils -y
mkdir /mnt/nas_share
```

创建密码文件（注意保护文件，此处为明文密码）：
```
nano ~/.smbcredentials
```

设置SMB登录密码，自行替换：
```
username=smb_share
password=share_password
```

修改自动挂载文件
```
nano /etc/fstab
```

加入挂载位置，自行替换
```
//$smb_server/share /mnt/nas_share cifs credentials=/root/.smbcredentials,iocharset=utf8 0 0
```

## 6、安装docker

一键安装Docker
```
apt install curl -y
curl -sSL https://get.daocloud.io/docker | sh
```

安装portainer
```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

## J4125 CPU使用Jellyfin，可能需要使用以下Docker影响
nyanmisaka/jellyfin:latest 

## N5105需要开启GuC，并更新PVE内核

开启GuC
```
nano /etc/default/grub

加入：
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on i915.enable_guc=3 quiet"

update-grub
reboot
```

确认是否开启
```
dmesg | grep -iE "guc|huc"
```

更新5.17内核
```
curl -1sLf 'https://dl.cloudsmith.io/public/pve-edge/kernel/gpg.8EC01CCF309B98E7.key' | apt-key add -
echo "deb https://dl.cloudsmith.io/public/pve-edge/kernel/deb/debian bullseye main" > /etc/apt/sources.list.d/pve-edge-kernel.list

apt update
apt install pve-kernel-5.17-edge -y
```