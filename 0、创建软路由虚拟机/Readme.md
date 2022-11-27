## 识别网口
1. 安装ethtool
```
apt install ethtool -y
```
2. 打开端口自动启动 & 重启系统
3. 确认所有网卡设备位置
```
lspci | grep -i 'eth'
```
4. 通过ethtool识别网口对应设备位置以及系统设备名
```
ethtool -i [设备名称]  #查看设备名对应设备位置
ethtool [设备名称]  #通过查看是否连接确认设备名对应实际网口，如果硬件支持可以使用ethtool --identify [设备名] 命令确认）
```
5. 关闭端口自动启动 & 重启系统

## 开启硬件直通 

#### BIOS中打开硬件直通相关选项（VT-d & VMX）
#### 编辑Grub
```nano /etc/default/grub```

#### 注释原条目，并增加开启参数
```GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"```

#### 如果你的pcie设备分组有问题也可以换成这一行对分组拆分（直通遇到问题都可以尝试这个）
```GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on pcie_acs_override=downstream"```

#### 更新grub
```update-grub```
## 上传启动镜像

将镜像img扩展名修改为iso，直接通过PVE后台上传
## 创建虚拟机并设置直通
修改配置文件命令
```
nano /etc/pve/qemu-server/[虚拟机编号].conf
```
