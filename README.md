# GearOS

#### 介绍
GearOS是由openEuler开源社区Industrial-Control SIG孵化的一款面向工业控制领域的实时增强操作系统，专注于操作系统实时性、可靠性，基于openEuler开源操作系统，使用Yocto构建，可应用于汽车控制、机器人控制、PLC控制、机床控制等领域。

本次发布的GearOS版本基于ARM64架构，主要包含两个内核和两个文件系统镜像。
两个内核：分别为支持Preempt_RT实时特性的内核和支持Xenomai实时特性的内核，均基于openEuler 4.19内核改造而来，大小为8MB。
两个文件系统镜像：分别为紧凑型文件系统镜像和标准文件系统镜像。其中紧凑型文件系统镜像使用BusyBox制作，大小为5.4MB；标准文件系统镜像未使用BusyBox。

#### 软件架构

![GearOS架构](https://images.gitee.com/uploads/images/2021/1105/170120_5f3cea00_5548898.png "屏幕截图.png")

#### GearOS技术特性

- 系统主要特性
支持飞腾2000/4、鲲鹏920、TI AM335X、Qemu-ARM64、X86等平台
内核最低可做到3.3MB，本次发布8MB
内核支持串口、网络、块设备、USB、PCIe等驱动
文件系统最低可做到5.4MB
启动时间小于5S
支持Preempt_RT和Xenomai实时方案
紧凑型文件系统镜像包含登录验证、Udev、SSH、Xenomai库、rt-tests工具集、
标准型文件系统镜像增加Python、Perl、OpenSSL、Sqlite、RPM包管理等

- 工业相关附加特性（已支持暂未集成）
支持LibModbus协议
支持EtherCAT协议
支持OPC UA协议
支持TSN
支持HSR/PRP
支持NETCONF/YANG

- 实时相关特性
在FT-2000/4、鲲鹏920硬件设备，使用openEuler 4.19内核，使用cyclictest测试工具对比测试结果


![实时性测试结果](https://images.gitee.com/uploads/images/2021/1105/170113_e42bc343_5548898.png "屏幕截图.png")

- 未来计划
支持树莓派4、NXP i.MX 7、瑞芯微RK3399等平台
支持5G、Bluetooth 、NFC、ZigBEE设备及相关协议
支持CoAP、MQTT等IOT相关协议
OTA
可靠性、安全性增强
虚拟化特性
实时性优化
CoDeSys运行时
IDE
其他嵌入式或工控需求

#### 使用说明

1.  vmlinuz-4.19-xenomai为Xenomai特性内核，支持ARM64架构下的飞腾2000/4，鲲鹏920
2.  vmlinuz-4.19-preempt-rt为Preempt_rt特性内核，支持ARM64架构下的飞腾2000/4，鲲鹏920
3.  initramfs-minimal.img为使用busybox制作的文件系统镜像
4.  initramfs.img为常规文件系统

#### 参与贡献

姓名	公司	Gitee ID	邮箱
郭皓	麒麟软件	guohaocs2c	guohao@kylinos.cn
马玉昆	麒麟软件	kylin-mayukun	mayukun@kylinos.cn
吴春光	麒麟软件	wuchunguang	wuchunguang@kylinos.cn
丁丽丽	麒麟软件	blueskycs2c	dinglili@kylinos.cn
张继文	麒麟软件	zhang-jiwen	zhangjiwen@kylinos.cn
张茜	麒麟软件	zxiiiii	zhangxi@kylinos.cn
黎亮	华为	liliang_euler	liliang889@huawei.com
张攀	华为	SuperHugePan	zhangpan26@huawei.com


