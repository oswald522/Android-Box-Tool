# 通用 Android Root 教程（完整版！附砖机自救方法）

获取 Android 设备的 Root 权限，是解锁设备高级功能的前提。本教程将指导您完成 Root 的基本流程，并解决在 Root 过程中可能遇到的权限和系统锁问题，最后提供砖机自救的终极方案。

## 一、 Root 权限的原理

理论上，获取 Root 权限只需在系统中放置并配置三个关键文件/组件：

1.  `/system/bin/su` (Root 命令)
2.  `/system/xbin/su` (部分软件识别的 Root 路径，可通过软链接实现)
3.  `/system/app/SuperUser.apk` (Root 权限管理器)

由于 `/system/bin/su` 和 `/system/xbin/su` 指向同一个文件，因此只需完成以下配置：

1.  **放置文件：** 放置 `/system/bin/su` 和 `/system/app/SuperUser.apk`。
2.  **创建软链接：** `ln -s /system/bin/su /system/xbin/su`
3.  **设置权限：** 必须设置 `/system/bin/su` 具有 SUID/SGID 权限，确保任何用户执行该命令时，都以 Root 身份运行：`chmod 4755 /system/bin/su`

## 二、 传统方法的局限性（为什么直接操作会失败）

虽然上述操作看起来很简单，但在未 Root 的设备上直接执行会遇到三大难题：

1.  **/system 路径是只读的：** 默认情况下，`/system` 分区是只读的，无法写入或修改。
2.  **`chmod` 需要 Root 权限：** 设置 SUID/SGID 权限（`chmod 4755`）需要 Root 权限才能执行，形成死循环。
3.  **系统自动恢复：** 部分系统在启动时会自动将 `/system/bin/su` 的权限改回 `755`，甚至直接删除该文件。

## 三、 绕过限制：利用 ADB “后门”

为了解决上述问题，我们需要利用 Google 留给开发者的“后门”工具：**Android Debug Bridge (ADB)**。

### 准备工作

1.  **连接设备：** 使用数据线将 Android 设备连接到 PC。
2.  **安装驱动：** 确保 PC 上已安装对应 Android 设备的驱动程序。
3.  **开启调试：** 在 Android 设备上进入 **设置 -> 应用程序 -> 开发**，勾选 **USB 调试 (USB debugging)**。
4.  **准备工具：** 下载并解压 Android SDK 中的 `adb` 工具包到 PC 上的一个文件夹（例如：`C:\adb_root`）。
5.  **准备 Root 文件：** 将附件中的 `su` 文件和 `SuperUser.apk` 复制到该工具包文件夹中。

### 执行 ADB Root 步骤

1.  **打开命令窗口：** 在工具包文件夹 (`C:\adb_root`) 中，按住 `Shift` 键，右键单击空白处，选择 **“在此处打开命令窗口”**。
2.  **执行命令序列：** 依次在命令窗口中输入以下指令（每次输入后回车）：

    ```bash
    adb remount
    adb push su /system/bin
    adb push SuperUser.apk /system/app
    adb shell ln -s /system/bin/su /system/xbin/su
    adb shell chmod 4755 /system/bin/su
    ```

3.  **重启设备：**
    ```bash
    adb reboot
    ```

### 遗留问题（进阶处理）

即使使用 ADB，仍可能遇到以下问题：

1.  **`adb remount` 报错：** 少数设备无法将 `/system` 重新挂载为可读写模式。
2.  **重启后权限丢失：** 重启后 SUID/SGID 权限被系统恢复或文件被删除。

这些问题的根源在于 **Boot 镜像（Boot Image）** 在系统启动内核时设置的“锁”。要彻底解决，就必须刷入修改过的第三方 Boot 镜像，这涉及到刷机操作。

## 四、 终极方案：刷机与砖机自救

Android 系统启动有三种模式：正常启动、Bootloader 模式、Recovery 模式。

### 1. 刷机入口（Bootloader 和 Recovery）

*   **Bootloader 模式（推荐用于救砖）：** 提供通过 USB 线刷机的接口。
*   **Recovery 模式：** 提供通过本地存储（如 SD 卡）刷机的方法。原版 Recovery 通常会验证刷机包的数字签名，限制第三方刷机包的安装。

### 2. 进入刷机模式

*   **通过 ADB 进入：**
    ```bash
    adb reboot bootloader   // 进入 Bootloader 模式
    adb reboot recovery     // 进入 Recovery 模式
    ```
*   **通过物理按键进入（最常用且保险的方法）：**
    1.  设备完全关机。
    2.  按住特定组合键（例如：**音量上键 + 电源键**，不同机型不同）同时开机。

### 3. 使用 Fastboot 刷机（Bootloader 模式）

Bootloader 模式允许我们使用 `fastboot` 工具线刷分区镜像。

**前提条件：** 找到与您设备型号对应的完整刷机包（包含 `boot.img`, `recovery.img`, `system.img` 等）。

1.  将所有 `.img` 镜像文件放入 `fastboot` 工具所在的文件夹。
2.  在命令窗口中执行刷写命令（**注意：刷写分区有风险，请确保镜像文件正确**）：

    ```bash
    fastboot flash boot boot.img        // 刷入新的内核/Boot
    fastboot flash recovery recovery.img  // 刷入第三方 Recovery
    fastboot flash system system.img     // 刷入系统
    fastboot flash userdata data.img     // 刷入用户数据（可选）
    fastboot reboot                     // 重启设备
    ```

### 4. 砖机自救指南

**只要设备还能进入 Bootloader 模式，它就不是“真砖”，仍有救回的可能。**

**自救流程：**

1.  通过物理按键（或 ADB）强制设备进入 **Bootloader 模式**。
2.  利用 Google 或百度搜索，找到您设备型号对应的官方或可靠的**完整系统镜像文件**（通常是一系列 `.img` 文件）。
3.  使用上述 **`fastboot flash`** 命令，将官方或可靠的镜像逐一刷入，覆盖掉导致问题的系统分区。
4.  刷写完成后，执行 `fastboot reboot` 尝试正常启动。