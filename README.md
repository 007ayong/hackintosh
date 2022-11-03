# 使用 OC 引导 AMD Ryzen 黑苹果

## 我的配置

- CPU：AMD Ryzen 7 2700X Eight-Core 八核
- 显卡：AMD Radeon RX 5500 XT (8 GB / 蓝宝石)
- 主板：微星 B450M PRO-VDH PLUS (MS-7A38) (LPC Controller B450芯片组)
- 内存：16 GB ( 威刚 DDR4 2666MHz )
- 声卡：板载 Realtek ALC892
- 麦克风：USB 耳麦
- 有线网卡：板载 Realtek RTL8111H 千兆网卡
- 无线网卡：BCM 94360 CD

主板详细参数来源：https://detail.zol.com.cn/1265/1264709/param.shtml



## OC 引导配置文件说明

### OC 版本及升级

当前使用 OpenCore 版本为 0.8.3 开发版 (支持 macOS 13 beta)。推荐使用可视化工具 [OCAT (全称：OpenCore Auxiliary Tools)](https://github.com/ic005k/OCAuxiliaryTools) 编辑、升级 OC 等。


### AMD 内核补丁

在 config.plist 中找到 `Kernel > Patch` 部分，可使用 [AMD_Vanilla](https://github.com/AMD-OSX/AMD_Vanilla) 的 patches.plist. 

但其中 Comment 中包含 algrey - Force cpuid_cores_per_package 的三个项目，按 CPU 核心数修改 Replace 值。具体修改方法参见 README 文档。

> 注意：需要先启用 `Kernel > Quirks` 中的 `ProvideCurrentCpuInfo`。


### Kernel 内核扩展

可使用 OCAT 更新，其中 AMDRyzenCPUPowerManagement.kext 和 SMCAMDProcessor.kext 是 AMD CPU 温度和频率控制的扩展。需要前往 [SMCAMDProcessor](https://github.com/trulyspinach/SMCAMDProcessor) 仓库下载最新版本。

- [Lilu.kext](https://github.com/acidanthera/Lilu)：基础性依赖
- [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC)：负责传感器相关
- [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen)：负责显卡 GPU 相关
- [AppleALC.kext](https://github.com/acidanthera/AppleALC)：负责 Realtek 音频相关
- [RealtekRTL8111.kext](https://github.com/Mieze/RTL8111_driver_for_OS_X)：负责有线网卡相关
- [AMDRyzenCPUPowerManagement.kext](https://github.com/trulyspinach/SMCAMDProcessor)：负责 CPU 频率
- [SMCAMDProcessor](https://github.com/trulyspinach/SMCAMDProcessor)：上一个扩展配套的传感器参数传递，同时也依赖于 VirtualSMC.kext
- [NVMeFix.kext](https://github.com/acidanthera/NVMeFix)：负责 NVMe 存储相关内容


> CPU 温度可能显示异常，macOS 13 beta 需最新版本否则报错。

### 清除 Nvram

OC 0.8.3 以后，清除 Nvram 改为了模块化，自定义添加。

在 config.plist 中找到 `UEFI > Drivers` 部分，添加 ResetNvramEntry.efi，在 [OpenCore](https://github.com/acidanthera/OpenCorePkg) 压缩包中，可在 Drivers 文件夹找到。

### 关闭 SIP

同 Nvram 部分，手动添加至 Drivers 即可，文件名为 ToggleSipEntry.efi

### 音频

从 [该页面](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs) 查找你的 Realtek 声卡型号，并记录 layout 后面的值，找到 config.plist 中 `NVRAN > Add >  7C436110-AB2A-4BBB-A880-FE41995C9F82 > boot-args`，添加参数 `alcid=16`，将 16 改成前面记录的 layout 值，每次添加一个 id 进行测试。

### 显卡

AMD Navi 10 Series GPUs (RX 5500, RX 5600, RX 5700) 需要在 `NVRAN > Add >  7C436110-AB2A-4BBB-A880-FE41995C9F82 > boot-args` 添加 `agdpmod=pikera` 解决黑屏问题。

### 网卡

板载的有线网卡进行内建处理：

使用 [Hackintool](https://github.com/headkaze/Hackintool) 获取有线网卡的 PCIe 设备地址，例如：`PciRoot(0x0)/Pci(0x1,0x3)/Pci(0x0,0x2)/Pci(0x4,0x0)/Pci(0x0,0x0)`，在 config.plist 中 `DeviceProperties` 中添加你的 PCIe 地址，并添加 ``built-in Data 01`。可解决 iCloud、App Store 登陆失败等问题。





