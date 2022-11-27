**（文中出现命令均在PVE网页端命令行Shell中进行）**
## PVE开启硬件直通功能（此处开始针对初次安装PVE用户，如根据前面教程操作到这里的可以不做）
1. BIOS中关闭CSM（兼容启动模式），仅开启UEFI启动，并以UEFI安装PVE
2. BIOS中打开硬件直通相关设置
找到BIOS中虚拟化功能，并打开（VMX或SVM)
找到BIOS中硬件直通相关功能，并打开（VT-d，AMD-v）
3. BIOS的CSM设置里面把vedio的UEFI换成legacy
4. 修改GRUB，启动IOMMU（硬件直通)支持
```
#修改grub启动文件
nano /etc/default/grub

#将以下条目删除
GRUB_CMDLINE_LINUX_DEFAULT="quiet"

#替换为
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt nofb nomodeset video=vesafb:off video=efifb:off video=simplefb:off pcie_acs_override=downstream"

#按“CTRL+X”退出修改，然后“Y”确认保存，然后按“enter”确认文件名并保存

#更新GRUB
update-grub

#pcie_acs_override （PCIE拆分，增加直通成功率）
#video=vesafb:off,efifb:off （关闭framebuffer，避免显卡占用）
```
## 加载直通所需要的Linux模组

```
#修改模组加载文件
nano /etc/modules

#添加以下内容
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd 
```

## 屏蔽显卡相关驱动

```
#创建屏蔽文件
nano /etc/modprobe.d/pve-blacklist.conf

#加入需要屏蔽的显卡相关模组（包含声卡以及i915核显驱动）
blacklist snd_hda_intel
blacklist snd_soc_skl
blacklist snd_sof_pci
blacklist snd_hda_codec_hdmi
blacklist i915
```

## 配置对应的直通相关模组，绑定设备

#确认显卡和声卡硬件ids
```
#查询设备位置
lspci

#查询对应设备ids
lspci -n -s 00:02.0
```

```
#创建设置文件
nano /etc/modprobe.d/igpu-pass.conf
```
```
#加入以下条目——————————

#配置vfio-pci，绑定设备，ids处输入前面查到的先看ids号
options vfio-pci ids=8086:3185,8086:3198 disable_vga=1

#避免ignored RDMSR警告刷屏
options kvm ignore_msrs=1 report_ignored_msrs=0

#允许设备不安全中断，增加成功率
options vfio_iommu_type1 allow_unsafe_interrupts=1

#激活声卡（不确定）
options snd-hda-intel enable_msi=1
```
# 更新配置并重启

```
update-initramfs -u -k all
reboot
```
# 下载显卡rom文件
```
#进入路径
cd /usr/share/kvm

#下载文件
wget https://gitee.com/spoto/PVE_Generic_AIO/raw/master/Z.1%E3%80%81J4125%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%B8%E6%98%BE%E7%9B%B4%E9%80%9A%EF%BC%88%E5%90%ABHDMI%E8%BE%93%E5%87%BA%EF%BC%89/vbios_1005v4.bin
```
# 创建虚拟机需要注意的地方
1. CPU使用host模式
2. q35机型直通成功率较高，亦可尝试i440fx
3. 直通PCI硬件（设置为主GPU，PCIE模式）
4. 删除原本虚拟显示设备

# 为核显加载BIOS文件
```
#xxx为虚拟机编号VMID
nano /etc/pve/qemu-server/xxx.conf 

#找到通过host硬件ID找到硬件描述的一行并加入
romfile=vbios_1005v4.bin
```

# 插入HDMI接口启动成功