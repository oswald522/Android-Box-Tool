# 手动安装 SuperSU 终极指南 (基于 ADB)

对于追求终极控制的资深用户和开发者，手动通过 ADB 安装 SuperSU 是一种绕过官方安装程序、确保 Root 文件正确的可靠方法。

**重要前提条件：**

在开始操作前，请确保您的目标设备满足以下两个关键条件：

1. **ADB 具备 Root 权限：** 您需要能够通过 ADB 成功执行 Root 命令，例如：
   ```bash
   adb root
   ```
2. **System 分区可写：** 必须能够将系统分区重新挂载为可读写模式 (Read-Write, RW)：
   ```bash
   adb remount
   # 或者更底层的命令（取决于设备）
   mount -o remount,rw -t auto /system
   ```

---

## 第一步：准备文件并推送到设备

请首先解压您从官方渠道下载的 SuperSU 安装包[Recovery V2.82 Flashable.zip](https://supersuroot.org/downloads/SuperSU-v2.82-201705271822.zip)。确保您能找到核心的 `su` 二进制文件。

**操作步骤：**

1. **推送 `su` 文件：** 将 `su` 文件推送到设备的目标系统目录下（通常是 `/system/xbin/`）。
   ```bash
   adb push su /system/xbin/su
   ```

2. **进入 Shell 环境并设置权限：**
   ```bash
   adb shell
   cd /system/xbin
   ```

3. **赋予最高执行权限：**
   ```bash
   chmod 777 /system/xbin/su
   ```

4. **创建守护进程备份（可选但推荐）：** 为保持兼容性，我们将 `su` 复制一份，并命名为守护进程的名称：
   ```bash
   cp -a /system/xbin/su /system/xbin/daemonsu
   ```

## 第二步：配置开机自启动脚本

要让 SuperSU 稳定运行并接管 Root 权限请求，我们需要确保其守护进程在系统启动时自动运行。通常，这个过程是通过修改特定的初始化脚本来实现的。

**查找目标脚本：**

常见的开机自启动脚本路径包括 `/etc/init.d/`、`/system/etc/` 或 `/system/bin/` 中，通常以 `.sh` 结尾。**本教程以最常见的 `/system/etc/install-recovery.sh` 为例。**
通常还有`/system/bin/init.xxx.sh`文件，主要功能是开机自动启动运行的文件。

**修改脚本内容：**

我们将添加命令，使 SELinux 临时禁用（如果需要）并启动 SuperSU 守护进程。

```bash
# 1. 临时禁用 SELinux 强制模式（如果系统默认开启）
echo "setenforce 0" >> /system/etc/install-recovery.sh

# 2. 启动 SuperSU 的后台守护进程 (daemonsu)
echo "/system/xbin/daemonsu -ad" >> /system/etc/install-recovery.sh
```

> **注意：** `daemonsu -ad` 命令用于以后台模式运行 Root 守护进程。

## 第三步：配置网络 ADB 调试（针对电视盒子等设备）

如果您正在操作的网络设备（如电视盒子）不方便插 USB 线，可以配置其通过 Wi-Fi 开启 ADB 连接。

在**同一个**开机脚本 (`/system/etc/install-recovery.sh`) 中添加以下内容：

```bash
# 启用网络 ADB 调试（默认端口 5555）
echo "setprop service.adb.tcp.port 5555" >> /system/etc/install-recovery.sh

# 重启 adbd 服务以使端口生效
echo "stop adbd" >> /system/etc/install-recovery.sh
echo "start adbd" >> /system/etc/install-recovery.sh
```

## 第四步：完成与验证

完成以上所有修改后，退出 ADB Shell 环境并重启设备：

```bash
exit
adb reboot
```

设备重启后，SuperSU 的核心文件已经就位，并且守护进程会在后台运行。您现在应该可以使用 `adb shell` 切换到 `su` 用户，或者在设备上测试 Root 管理应用（如 Root Checker）是否能正确响应 SuperSU 的授权请求。