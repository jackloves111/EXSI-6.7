# ESXi 6.7 知识库

常见问题解决方案汇总

---

## 📋 目录导航

- [许可证激活](#一esxi-67-许可证激活)
- [虚拟机自动启动](#二esxi-67-70-虚拟机自动启动无效)
- [强制直通USB芯片组](#三esxi-67-70-强制直通板载usb芯片组)
- [重装系统保留数据](#四esxi-67-重装系统保留虚拟机数据)

---

## 一、ESXi 6.7 许可证激活

### 操作步骤

打开 WEBUI，进入 **管理 → 许可 → 分配许可证**

![许可证分配界面](images/csdn_3.png)

> ⚠️ **可用许可证秘钥：**
> 
> ```
> HV4WC-01087-1ZJ48-031XP-9A843
> NF0F3-402E3-MZR80-083QP-3CKM2
> 4F6FX-2W197-8ZKZ9-Y31ZM-1C3LZ
> JZ2E9-6D2DK-XZQD0-632E4-33E7Z
> MZ48M-DNK56-ZZJD0-RTCE2-9321X
> 0Y0AJ-4P29H-LZV81-59AQ2-C291V
> ```

<p align="right"><i>来源：<a href="https://blog.csdn.net/xiaobo57/article/details/138789335">CSDN - xiaobo57</a></i></p>

---

## 二、ESXi 6.7-7.0 虚拟机自动启动无效

### 问题描述

在 ESXi 中对虚拟机设置自启动后，重启服务器虚拟机不会自动启动。

![自启动设置界面](images/csdn_1.png)

### 解决方案

除了在虚拟机上设置自启动外，还需要在主机级别进行配置：

> ℹ️ **配置路径：** 【主机】→【管理】→【系统】→【自动启动】
> 
> 将自动启动设置修改为 **【是】** 即可。

![主机自动启动配置](images/csdn_2.png)

<p align="right"><i>来源：<a href="https://blog.csdn.net/fanfuqiang/article/details/116562467">CSDN - fanfuqiang</a></i></p>

---

## 三、ESXi 6.7-7.0 强制直通板载USB芯片组

### 背景

某些主板的板载USB芯片组在 ESXi 中显示为灰色状态，无法直接开启直通。以下方法可以强制开启直通功能。

![USB设备直通状态](images/csdn_4.png)

### 第一步：记录设备ID

在设备管理器中查看要直通的USB设备，记录以下信息：
- 供应商ID（Vendor ID）
- 设备ID（Device ID）

![设备ID查看](images/csdn_5.png)

### 第二步：开启ESXi SSH功能

通过 WEB 界面开启 SSH 服务，以便远程连接进行配置。

![SSH设置](images/csdn_6.png)

### 第三步：验证ID（可选）

通过 SSH 连接后，使用以下命令验证：

```bash
lspci -v | grep "Class 0c03"
```

其中 0C03 就是类ID。

![命令验证](images/csdn_7.png)

![验证结果](images/csdn_8.png)

### 第四步：添加直通配置

编辑直通配置文件：

```bash
vi /etc/vmware/passthru.map
```

在文件最后添加以下内容：

```
# Intel Corporation 8 Series/C220 Series Chipset Family USB xHCI
8086  8C31  d3d0     default

# 其中：
# 8086 = 供应商ID
# 8C31 = 设备ID
# d3d0 和 default 为固定值
```

![配置文件编辑](images/csdn_9.png)

> ⚠️ **注意：** 保存退出后需要重启 ESXi 才能生效！

```
按 Esc 键
输入 :wq 回车保存退出
```

重启完成后，即可在直通设置中看到该USB设备可以正常切换直通。

![直通设置成功](images/csdn_10.png)

<p align="right"><i>来源：<a href="https://blog.csdn.net/y59724555/article/details/118014879">CSDN - 济源市天源科技有限公司</a></i></p>

---

## 四、ESXi 6.7 重装系统保留虚拟机数据

### 背景

当 ESXi 系统出现故障或需要重置密码时，可以通过重新安装系统来恢复，且不会丢失虚拟机数据。

> ℹ️ **适用场景：**
> - ESXi 系统故障无法启动
> - 忘记管理员密码需要重置
> - 需要升级系统版本

### 测试环境

- 服务器 1 台
- ESXi 6.7.0 安装镜像
- CentOS 虚拟机 1 台（用于验证数据）
- 测试文件：123.txt（内容 "hello word"）

### 第一步：挂载安装镜像

将 ESXi 安装 ISO 文件挂载到服务器的虚拟光驱。

### 第二步：修改BIOS启动项

进入服务器 BIOS 界面，将第一启动项设置为虚拟 CD-ROM。

![BIOS设置](images/cnblog_1.png)

![启动项设置](images/cnblog_2.png)

![保存退出](images/cnblog_3.png)

### 第三步：重新安装ESXi

按照正常流程安装 ESXi 操作系统。

![安装界面](images/cnblog_4.png)

![安装选项](images/cnblog_5.png)

![选择磁盘](images/cnblog_6.png)

![安装进度](images/cnblog_7.png)

![安装完成](images/cnblog_8.png)

![重启提示](images/cnblog_9.png)

### 第四步：选择安装选项（关键步骤）

> ⚠️ **重要：** 在选择安装硬盘后，会出现三个选项：
> 
> 1. 升级 ESXi 操作系统，保存 VMFS 数据存储
> 2. **安装 ESXi 操作系统，保存 VMFS 数据存储 ✅**
> 3. 安装 ESXi 操作系统，覆盖安装 VMFS（数据会丢失！）
> 
> 使用空格键选择**第二个选项**，然后按回车确认。

![安装选项](images/cnblog_10.png)

![选择确认](images/cnblog_11.png)

![安装过程](images/cnblog_12.png)

![安装完成](images/cnblog_13.png)

### 第五步：登录并重新注册虚拟机

系统安装完成后，登录 WEB 界面。

![登录界面](images/cnblog_14.png)

![WEB界面](images/cnblog_15.png)

此时虚拟机列表为空，需要重新注册。

![空虚拟机列表](images/cnblog_16.png)

点击 **虚拟机 → 创建/注册虚拟机**

![注册虚拟机](images/cnblog_17.png)

选择 **"注册现有虚拟机"**，浏览数据存储中的虚拟机文件。

![选择虚拟机](images/cnblog_18.png)

![浏览存储](images/cnblog_19.png)

![选择文件](images/cnblog_20.png)

![确认注册](images/cnblog_21.png)

### 第六步：验证数据完整性

启动虚拟机，检查测试文件是否丢失。

![虚拟机启动](images/cnblog_22.png)

![系统启动](images/cnblog_23.png)

> ℹ️ **验证结果：** 虚拟机正常启动，123.txt 文件无丢失！✅

![数据验证](images/cnblog_24.png)

<p align="right"><i>来源：<a href="https://www.cnblogs.com/Koji/p/14442730.html">博客园 - KoJi-FJY</a></i></p>

---

<p align="center"><i>ESXi 6.7 知识库 | 整理于 2024年</i></p>
