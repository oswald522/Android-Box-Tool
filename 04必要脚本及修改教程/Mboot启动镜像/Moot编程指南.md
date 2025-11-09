好的，这是对您提供的 **MBoot User Guide** 的翻译：

# MBoot 用户指南

## 修订历史

| 文档版本 | 日期 | 作者 | 备注 |
| :--- | :--- | :--- | :--- |
| V 0.1 | 2009/04/16 | Nick.Lai | 创建 |
| V 0.2 | 2009/05/05 | Terry.Tsai | 更新 |
| V 0.3 | 2009/07/16 | CM.Chen | 更新 |
| V 0.4 | 2010/05/31 | CP.Hsu | 添加块头格式 (Chunk Header Format) |
| V 0.5 | 2011/01/14 | CP.Hsu | 添加 SBoot 和 UBoot 的版本规则 |
| V 0.6 | 2011/01/19 | Cyber.Chang | 添加 SPI 闪存布局 |
| V 0.7 | 2011/03/26 | Eric.Peng, Cyber.Chang | 添加 MIU 设置 / 环境变量 |
| V 0.8 | 2011/11/29 | Cyber.Chang | 添加寄存器地址转换 |
| V0.9 | 2012/02/08 | ERIC.Wu | 更新块头状态 |
| V1.0 | 2012/10/8 | Eric.Wu | u-boot-2011.06 的新架构 |

## 缩略词

| 缩略词 | 描述 |
| :--- | :--- |
| MBoot | Mstar 引导加载程序 (Mstar Bootloader) |
| SBoot | 小型引导加载程序 (Small Bootloader) |
| UBoot | 通用引导加载程序 (Universal Bootloader) |

## 目录

1. MBoot ................................................................................... 1
    1.1 MBoot 概述 ........................................................................ 1
    1.2 MBoot 源码树 .................................................................... 2
    1.3 如何编译 MBoot ................................................................ 3
    1.4 如何使用 MBoot，并通过 tftp 服务器烧写程序 ....................... 6
    1.5 SPI 闪存布局 .................................................................... 7
2. SBoot ..................................................................................... 8
    2.1 SBoot 概述 ........................................................................ 8
    2.2 SBoot 设置 ....................................................................... 9
    2.3 SBoot 启动流程 .............................................................. 11
    2.5 SBoot 版本规则 .............................................................. 13
    2.6 MIU 设置 ........................................................................ 15
    2.7 寄存器地址转换 .............................................................. 17
3. 块头格式 (Chunk Header Format) ………………………………………………
4. UBoot .................................................................................... 18
    4.1 u-boot-1.1.6 概述 ........................................................... 18
    4.2 u-boot-2011.06 概述 ....................................................... 19
    4.3 自动启动序列 ................................................................ 20
    4.4 UBoot 版本规则 .............................................................. 20
    4.5 环境变量 ...................................................................... 23
5. uboot-2011.06 的新架构

***

## 1. MBoot

本章将介绍 MBoot，包括概述、代码树、编译流程和环境设置。

### 1.1 MBoot 概述

MBoot 是 MStar 的引导加载程序，由 `pm.bin`、`sboot.bin` 和 `uboot.bin` 组成。MBoot 用于启动系统，它会初始化硬件设置，然后从 NAND 闪存加载 Linux 内核和应用程序到 DRAM。除了启动系统，MBoot 还负责软件升级（OTA/网络/USB）、显示启动 Logo 和播放启动音乐。

### 1.2 MBoot 源码树

MBoot 的源代码位于 `//DAILEO/MBoot`。

`pm.bin` 存储在此路径中。

在源码树中，可以看到两个与 u-boot 相关的文件夹。“u-boot-1.1.6” 用于 MStar 的旧 SOC，“u-boot-2011.06” 用于 MStar 的新 SOC。由于 uboot 是开源代码，我们希望将 MStar 的代码与开源代码分开，以便于维护。因此，我们为 u-boot-2011.06 创建了三个文件夹：“MstarApp”、“MstarCore” 和 “MstarCustomer”。在这三个文件夹中，所有功能都是由 MStar 的工程师设计的，并且这些功能都基于 u-boot-2011.06。“sboot” 文件夹用于 `sboot.bin`。`sboot.bin` 的主要任务是底层硬件初始化。`pm.bin` 以二进制格式存在于此源码树中，所有二进制文件都存储在 `sboot/bin/pm` 中。

### 1.3 如何编译 MBoot

以下 MBoot 的编译流程展示了如何生成 MBoot bin 文件。

* 将工作目录更改为 `MBoot/sboot`
* 运行 `“make menuconfig”` 来设置 SBoot 配置菜单
* 选择并进入 “Platform Configuration”（平台配置）
* 选择芯片（Chip）、目标板（Target board）和内存映射类型（Memory Map type）然后退出
* 选择并进入 “Module Options”（模块选项）
* 选择要包含到 UBoot 中的模块
* 选择并进入 “Save Configuration to an Alternate File”（将配置保存到备用文件）
  * 将配置文件保存为 `.config`
* 退出 SBoot 配置菜单
* 运行 `“make”` 来编译 UBoot 和 SBoot 源代码，然后将它们的 bin 文件合并成 MBoot bin 文件。
* MBoot bin 文件将生成在 `“MBoot/sboot/out/mboot.bin”`
* 使用 ISP 工具将 MBoot bin 文件烧录到目标板的 SPI 闪存中

### 1.4 如何使用 MBoot，并通过 tftp 服务器烧写程序

目标板重启后，控制台会显示提示符 `“<< MStar >>#”`。此时，我们可以在控制台上输入一些命令与 UBoot 进行交互。

为了通过以太网接口加载 Linux 内核和应用程序镜像（尤其是在开发/调试阶段），需要 TFTP 服务器。应为 LAN 设置设置以下环境变量：

* `setenv ipaddr 172.16.89.199` (基于您的本地 IP 地址，并将第三个字段加 1。)
* `setenv serverip 172.16.88.60` (您的本地 IP 地址)
* `macaddr 00 XX YY 00 00 01` (XXYY 是您的分机号。)
* `saveenv` (将环境保存到 NAND 闪存)

在控制台上键入 `“mstar”` 命令以下载 Linux 内核和应用程序到 DRAM，然后写入 NAND 闪存。

### 1.5 SPI 闪存布局

目前 MStar 的电视机有两种启动模式，因此我们有两种用于 SPI 的镜像布局。这两种启动模式是“维护启动 (Housekeeping booting)”和“PM51 启动 (PM51 booting)”。

**维护启动模式 (Housekeeping booting mode):**

| 偏移量 (offset) | 代码 (Code) | 大小 (size) |
| :--- | :--- | :--- |
| 0x00000 | sboot.bin | 0x10000 |
| 0x10000 | pm.bin | 0x10000 |
| 0x20000 | ChunkHeader | 0x80 |
| 0x20400 | u-boot.bin | |
| ... | ... | ... |
| 最后一个第 7 个块 (Last 7th bank) | bootlog + 启动音乐 | 0x40000 |
| ... | ... | ... |
| 最后一个第 3 个块 (Last 3th bank) | nand 镜像签名 | 0x10000 |
| 最后一个第 2 个块 (Last 2th bank) | nand 镜像签名 | 0x10000 |
| 最后一个第 1 个块 (Last 1th bank) | ENV1 | 0x10000 |
| 最后一个第 0 个块 (Last 0th bank) | ENV2 | 0x10000 |

**PM51 启动模式 (PM51 booting mode):**

| 偏移量 (offset) | 代码 (Code) | 大小 (size) |
| :--- | :--- | :--- |
| 0x00000 | pm.bin | 0x10000 |
| 0x10000 | sboot.bin | 0x10000 |
| 0x20000 | Customer Key bank | 0x10000 |
| 0x30000 | ChunkHeader | 0x80 |
| 0x30400 | u-boot.bin | |
| ... | ... | ... |
| 最后一个第 7 个块 (Last 7th bank) | bootlog + 启动音乐 | 0x40000 |
| ... | ... | ... |
| 最后一个第 3 个块 (Last 3th bank) | nand 镜像签名 | 0x10000 |
| 最后一个第 2 个块 (Last 2th bank) | nand 镜像签名 | 0x10000 |
| 最后一个第 1 个块 (Last 1th bank) | ENV1 | 0x10000 |
| 最后一个第 0 个块 (Last 0th bank) | ENV2 | 0x10000 |

实际上，PM51 启动模式仅用于安全启动系统。因此，如果听到“安全启动”，其镜像布局与“PM51 启动”模式相同。“安全启动”不是本文档的主题。如果您想了解更多相关信息，请查阅其他相关文档。

## 2. SBoot

SBoot（小型引导加载程序）是系统启动入口，用于初始化 CPU、缓存、硬件寄存器等。完成所有设置后，它将跳转到 UBoot 入口。

### 2.1 SBoot 概述

`sboot` 目录包含以下子目录：`bin`、`inc`、`scripts`、`out` 和 `src`。

* **bin**: UBoot 输出的 bin 文件和内存映射。
* **inc**:
    1. 板定义：在 `MBoot\sboot\inc\xxx\board` 中添加或修改板定义。
    2. 内存映射：在 `MBoot\sboot\inc\xxx\board\mmap` 中添加或修改内存映射。
* **scripts**: 用于菜单配置窗口的脚本文件。
* **out**: MBoot 输出的 bin 文件。
* **src**: 与芯片驱动相关的 SBoot 源代码文件。

### 2.2 SBoot 设置

* **SBoot 环境设置**:
  * 芯片 (Chip)：`MBoot\sboot\inc\titania2\board\chip`
  * 板 (Board)：`MBoot\sboot\inc\titania2\board`
  * 内存映射 (Mmap)：`MBoot\sboot\inc\titania2\board\mmap`

* **sboot.lds 文件中的启动入口和地址**:
  * `Entry(BOOT_Entry)`
  * 起始地址：`0xBFC00000`
  * `boot.o` 应放置在第一个 section，其代码大小不能超过 3K 字节。

    ```c
    MEMORY
    {
        boot : ORIGIN = 0xBFC00000,        LENGTH = 3K  
        rom : ORIGIN = 0x94000000+0xC00,  LENGTH = 8K
        ram : ORIGIN = 0x80200000,   LENGTH = 128K
        sram : ORIGIN = 0x84000000,   LENGTH = 1K
    }

    SECTIONS
    {
        .text1 :
        {
            KEEP(*boot.o(.text*))
        } > boot
        .text2 : AT ( LOADADDR(.text1) + SIZEOF(.text1) )
     ……
    }
    ```

* **启用 SBoot 缓存**:
  * 调用 `bal BOOT_InitCache`

* **硬件寄存器设置**:
  * `MBoot\sboot\src\xxx\bootrom.c`

* **设置 CPU 时钟、UART 波特率**:
  * `MBoot\sboot\src\xxx\boot.inc`

* **跳转到 UBoot**:
  * `MBoot\sboot\src\xxx\bootram.s`
  * `BOOT_CopyBootRAM` (将 BootRAM 拷贝到 DRAM) $\rightarrow$ `BOOTRAM_Entry` $\rightarrow$ UBoot 入口

### 2.3 SBoot 启动流程

SBoot 的流程图如下所示。SBoot 从 SPI 闪存的 `0xBFC00000` 开始，初始化硬件设置，利用 DSPRAM 空间执行一些硬件设置的 C 代码函数，将 `bootram` 段从 SPI 闪存复制到 DRAM 以便将 UBoot 从 SPI 闪存复制到 DRAM，最后跳转到 UBoot 入口。

### 2.4 SBoot 版本规则

* **如何设置 SBoot 版本**:
    在编译 `sboot.bin` 之前，将 SBoot 的变更列表号设置在以下位置。

    ```c
    //MBoot/sboot/src/version.h
    //-------------------------------------------------------------------------------------------------
    // Version Control
    //-------------------------------------------------------------------------------------------------
    #define MSIF_TAG                    {'M','S','V','C'}                   // MSVC
    #define MSIF_CLASS                  {'0','0'}                           // DRV/API (DDI)
    #define MSIF_CUS                    {'0','0','S','3'}
    #define MSIF_QUALITY                0
    #define MSIF_MOD                    {'S','B','T','_'}
    #define MSIF_DATE                   {'Y','Y','M','M','D','D'}
    #define MSIF_SBT_CHANGELIST         {'0','0','2','8','4','7','1','2'}    //P4 ChangeList Number

    #define SBT_VER                  /* Character String for SBOOT version               */  \
        MSIF_TAG,                       /* 'MSIF'                                           */  \
        MSIF_CLASS,                     /* '00'                                             */  \
        MSIF_CUS,                       /* '00S0'                                           */  \
        MSIF_QUALITY,                   /* 0                                                */  \
        MSIF_MOD,                       /* 'SBT_'                                           */  \
        MSIF_DATE,                      /* 'TTMMDD'                                         */  \
        MSIF_SBT_CHANGELIST,            /* CL#                                              */  \
        {'0','0','0'}

    typedef union _MSIF_Version
    {
        struct _DDI
        {
            U8                       tag[4];
            U8                       type[2];
            U16                       customer;
            U16                       model;
            U16                       chip;
            U8                       cpu;
            U8                       name[4];
            U8                       version[2];
            U8                       build[2];
            U8                       change[8];
            U8                       os;
        } DDI;
        struct _MW
        {
            U8                                     tag[4];
            U8                                     type[2];
            U16                                    customer;
            U16                                    mod;
            U16                                    chip;
            U8                                     cpu;
            U8                                     name[4];
            U8                                     version[2];
            U8                                     build[2];
            U8                                     changelist[8];
            U8                                     os;
        } MW;
        struct _APP
        {
            U8                                     tag[4];
            U8                                     type[2];
            U8                                     id[4];
            U8                                     quality;
            U8                                     version[4];
            U8                                     time[6];
            U8                                     changelist[8];
            U8                                     reserve[3];
        } APP;
    } MSIF_Version;

    const MSIF_Version _sbt_version = {
        .APP = { SBT_VER }
    };
    ```

* **如何在主应用程序中获取 SBoot 版本**:
    读取 `sboot.bin` 的最后 32 字节。（`sboot.bin` 将被离线可执行工具 `pad_version` 填充到 64K 字节，SBoot 版本信息存储在这 64K 字节的最后 32 字节中。）

    ```c
    //MBoot/sboot/Makefile
        out/sboot.bin: out/sboot.elf
     $(Q)$(OBJCOPY) -O binary -I elf32-little $< $@
     $(Q)$(HOSTCC) -o ./scripts/pad_version ./scripts/pad_version.c
     $(Q)./scripts/pad_version $@
    ```

### 2.5 MIU 设置

* **MIU 初始化流程**

您可以配置 MIU 的 3 个设置：

    *   **内存频率 (Memory frequency)**
        *   MIU0：运行 `make menuconfig` $\rightarrow$ Platform Configuration $\rightarrow$ Memory Frequency Selection
        *   MIU1：修改板配置文件中的 `MIU1_DRAM_FREQ`。例如，在 `BD_MST008B_10ATX_10405.h` 中。

    *   **内存大小 (Memory size)**
        *   运行 `make menuconfig` $\rightarrow$ Platform Configuration $\rightarrow$ Memory Map Type Selection
        *   **注意**: 如果出现启动问题，请咨询 CAE 工程师是否需要修改 `MIU0_DDR_Init[]` 或 `MIU1_DDR_Init[]`。例如，MIU0 的 `_RV32_2( 0x101202, 0x03a3 )`，MIU1 的 `_RV32_2( 0x100602, 0x02a2 )`。

    *   **MIU 接口 (MIU interface)**
        *   在板配置文件中，设置 `MIU_INTERFACE / MIU1_INTERFACE` 为 `DDR2_INTERFACE_BGA` 或 `DDR3_INTERFACE_BGA`。

* **在板定义中配置内存设置的示例**
    例如：`//MBoot/sboot/inc/titania13/board/BD_MST008B_10ATX_10405.h`
    您应调整 MIU 类型（DDR3 或 DDR2）和 MIU 频率如下：

    ```c
    //------Memory Setting----------------------------------------------------------
    #define BOOTUP_MIU_BIST                 1
    #ifndef MEMORY_MAP
    #define MEMORY_MAP                      MMAP_128_128MB//MMAP_64MB
    #endif
    #define MIU_INTERFACE                   DDR3_INTERFACE_BGA   //DDR3_INTERFACE_BGA

    #define MIU1_INTERFACE                  DDR2_INTERFACE_BGA   //DDR3_INTERFACE_BGA
    #define MIU1_DRAM_FREQ                  800 //950 / 1066 / 1300 / 1600

    #define ENABLE_AUTO_PHASE
    //------Memory Setting----------------------------------------------------------
    ```

    *p.s. MMAP 取决于 .config 文件，而不是此处。*

* **查找对应板的 MIU 头文件**:
    例如：`//MBoot/sboot/src/titania13/include/MIU_MST008B_10ATX_10405.h`

    现在您可以获取 CAE 提供的 MIU 设置表，并修改寄存器的值。例如：

    ```c
    _RV32_2( 0x101204, 0x000A ),  //如果 I64Mode =0x8b 否则 =0x0b
    _RV32_2( 0x101206, 0x0434 ),  //刷新周期=0x50  
    _RV32_2( 0x101208, 0x1899 ),  //reg_tRCD
    _RV32_2( 0x10120a, 0x2155 ),  //reg_tRRD
    _RV32_2( 0x10120c, 0x95a8 ),  //reg_tWL
    _RV32_2( 0x10120e, 0x406b ),  //tRFC
    _RV32_2( 0x101210, 0x1a50 ),  //MR0
    ```

### 2.7 寄存器地址转换

* **MIPS**
    `[RIU 偏移地址 (16位) X 2 + 块地址] X 2 + 0XBF000000`

* **ARM**
    `[RIU 偏移地址 (16位) X 2 + 块地址] X 2 + ARM RIU 起始地址`

* **8051**
    `RIU 偏移地址 (16位) X 2 + 块地址`

## 3. 块头格式 (Chunk Header Format)

`sboot.bin` 使用块头中保存的一些信息来跳转到下一个应用程序（对于 Linux 平台，下一个应用程序是指 UBoot；对于非 OS 平台，是指 Chakra2 主应用程序）。

块头格式如下所示。块头的基础地址取决于启动模式。如果启动模式是 “BOOTING\_FROM\_OTP\_WITH\_PM51” 或 “BOOTING\_FROM\_EXT\_SPI\_WITH\_PM51”，则基础地址为 `0x30000`。如果启动模式是 “BOOTING\_FROM\_EXT\_SPI\_WITH\_CPU”，则基础地址为 `0x20000`。

| | 0x00 | 0x04 | 0x08 | 0x0C |
| :--- | :--- | :--- | :--- | :--- |
| **0x00** | UBOOT\_ROM\_START | UBOOT\_RAM\_START | UBOOT\_RAM\_END | UBOOT\_ROM\_END |
| **0x10** | UBOOT\_RAM\_ENTRY | Reserved1 | Reserved2 | BINARY\_ID |
| **0x20** | LOGO\_ROM\_OFFSET | LOGO\_ SIZE | SBOOT\_ROM\_OFFSET | SBOOT\_ SIZE |
| **0x30** | SBOOT\_RAM\_OFFSET | PM\_ROM\_OFFSET | PM\_SIZE | PM\_RAM\_OFFSET |
| **0x40** | SECURITY\_INFO \_LOADER\_ROM\_OFFSET | SECURITY\_INFO \_LOADER\_SIZE | SECURITY\_INFO \_LOADER\_RAM\_OFFSET | CUSTOMER\_KEY\_BANK\_ROM\_OFFSET |
| **0x50** | CUSTOMER\_KEY\_BANK\_SIZE | CUSTOMER\_KEY\_BANK\_RAM\_OFFSET | SECURITY\_INFO \_AP\_ROM\_OFFSET | SECURITY\_INFO \_AP\_SIZE |
| **0x60** | UBOOT\_ENVIRONMENT\_ROM\_OFFSET | UBOOT\_ENVIRONMENT\_SIZE | DDR\_BACKUP\_TABLE\_ROM\_OFFSET | POWER\_SEQUENCE\_TABLE\_ROM\_OFFSET |
| **0x70** | UBOOT\_POOL\_ROM\_OFFSET | UBOOT\_POOLSIZE | | |
| **0x80** | | | | |

## 4. UBoot

UBoot（通用引导加载程序）是 DENX 开发的在 GPL 许可下的免费开源软件。

### 4.1 u-boot-1.1.6 概述

MStar UBoot 基于 u-boot 版本 1.1.6，它使用简单的命令行界面 (CLI) 与用户交互，通常通过串口控制台端口。

UBoot 应用了以下特性：

* 接受控制台命令来设置环境变量和执行系统操作。
* 初始化系统。
* 通过以太网接口加载内核和应用程序到 DRAM。
* 将内核和应用程序写入 NAND 闪存。
* 设置内核参数。
* 解压缩内核。
* 将启动参数传递给内核以启动。

`u-boot-1.1.6` 目录包含以下子目录：`board`、`common`、`cpu`、`disk`、`doc`、`drivers`、`fs`、`include`、`lib_generic`、`lib_mips`、`net`、`post`、`rtc` 和 `tools`。

| 目录 | 描述 |
| :--- | :--- |
| Board | 板定义 |
| common | 命令和环境设置 |
| Cpu | MIPS cpu start.s |
| Disk | 向用户报告设备信息 |
| Doc | 文档 |
| drivers | NAND 驱动、emac 驱动、usb 驱动等... |
| Fs | ext2, fat, fdos, jffs2, ... |
| include | 头文件 |
| lib\_generic | 通用库 |
| lib\_mips | MIPS 库 |
| net | 网络、tftp 等... |
| post | 上电自检 |
| rtc | 实时时钟 |

#### 4.1.2 UBoot 镜像

* **镜像布局**:
  * 入口点是 `vectors.S` 中的 `_start` @`0x875F0180`
  * `.u_boot_cmd`：所有命令过程都放在这个 section 中

### 4.3 自动启动序列

运行带 “bootcmd” 的命令

* `bootcmd=nand read.e 81000000 KL 300000; bootm 81000000;`
    1. 从 NAND 闪存读取内核镜像
    2. 调用函数 - `do_bootm()`
        * 从 `0x81000000` 加载镜像头 (image header)
        * 获取镜像真实数据地址 (`entry - header offset`)
        * 将镜像解压缩到数据加载空间 (`hdr->ih_load`)
        * 通过命令 `“cpmsbin”` 从 NAND 闪存读取 MStar bin 文件
    3. 调用函数 - `do_bootm_linux()`
        * 获取内核入口点 (`hdr->ih_ep`)
        * 初始化 Linux 参数:
            * `“Bootargs”` 是 Linux 内核所需的参数。
            * 这些参数放在 “Boot parameters section”。
        * 初始化环境设置:
            * 内存大小、initrd 地址和大小、闪存地址和大小
        * 跳转到内核入口点以启动 Linux 内核代码

### 4.4 UBoot 版本规则

* **如何设置 UBoot 版本**:
    在编译 `mboot.bin` 之前，将 UBoot 的变更列表号设置在以下位置。

    ```c
    //MBoot/u-boot-1.1.6/include/ms_ver.h
    static char MBOOT_VBuf[32] = {'M', 'S', 'V', 'C', '0', '0',
                           'B', '0',
                           BuildNum0,
                           BuildNum1,
                           BuildNum2,
                           BuildNum3,
                           BuildNum4,
                           BuildNum5,
                           '0', '0', '2', '0', '8', '7', '6', '8',
                           'T', 'H', '0', '0', '0', '0', '0', '0', '0',
                           'T'};
    ```

* **如何在主应用程序中获取 UBoot 版本**:
    调用 `prom_getenv("MBoot_version")` 来获取存储在环境变量中的 UBoot 版本。

    ```c
    //MBoot/u-boot-1.1.6/lib/lib_mips/mips_linux.c
         /*send MBoot version to linux kernel by seting environment*/
         /*please use prom_getenv("nev_name") function to get this information (e.g. prom_getenv("MBoot_version");)*/
         /*prom_getenv() define in \\RedLion\2.6.28.9\arch\mips\mips-boards\generic\init.c */
         memcpy(MBoot_Ver,MBOOT_VBuf,32);
         MBoot_Ver[32]='\0';
         linux_env_set ("MBoot_version", MBoot_Ver);
    ```

### 4.5 环境变量

您可以使用 UBoot 环境变量来存储程序设置。它们将存储在非易失性内存（SPI 闪存或 NAND 闪存）中。您可以通过运行 `make menuconfig` $\rightarrow$ Module Options $\rightarrow$ Env Config 进行配置。

* **管理环境变量的 UBoot 命令**
  * `printenv`: 打印所有环境变量的值。
  * `setenv`: 将环境变量 'name' 设置为 'value ...'。
  * `saveenv`: 将环境变量更新到非易失性内存。

* **环境变量 - bootcmd**:
    定义在 UBoot 阶段结束时要执行的命令。通常是启动 Linux 内核的命令。

* **环境变量 - bootargs**:
    定义传递给 Linux 内核的启动参数。内核将根据这些参数更改其行为。例如，修改控制台可以启用/禁用内核启动后的打印消息。
  * **启用打印消息**:
        `setenv  bootargs console= ttyS0,115200 ubi.mtd=3,2048 root=ubi:RFS rootfstype=ubifs ro LX_MEM=0x2000000 EMAC_MEM=0x100000 DRAM_LEN=0x10000000 LX_MEM2=0x66381000,0x1C00000 BB_ADDR=0x7FFF000 mtdparts=edb64M-nand:256k(NPT),256k(KL_BP),5m(KL),121m(UBI),-(NA)`
  * **禁用打印消息**:
        `setenv  bootargs console=/dev/null ubi.mtd=3,2048 root=ubi:RFS rootfstype=ubifs ro LX_MEM=0x2000000 EMAC_MEM=0x100000 DRAM_LEN=0x10000000 LX_MEM2=0x66381000,0x1C00000 BB_ADDR=0x7FFF000 mtdparts=edb64M-nand:256k(NPT),256k(KL_BP),5m(KL),121m(UBI),-(NA)`

* **环境变量 - bootdelay**:
    定义在执行 `bootcmd` 之前等待的秒数。
    例如：`setenv bootdelay 0`
    例如：`setenv bootdelay 3`

* **环境变量 - UARTOnOff**:
    定义 UART 是开启还是关闭。（默认开启）
    例如：`setenv UARTOnOff on`
    例如：`setenv UARTOnOff off`

## 5. uboot-2011.06 的新架构

### 5.1 三个文件夹的描述

为了便于维护，我们为 MStar 和客户功能创建了三个文件夹。

“MstarCore” 用于驱动，“MstarApp” 用于应用程序（例如，软件升级、OSD、显示启动 Logo 和启动音乐等）。“MstarCustomer” 用于定制化。这三个文件夹中的所有功能都基于 u-boot-2011.06。

### 5.2 入口点

现在有三个入口点可以将我们的功能添加到 uboot 的启动流程中。所有 MStar 或客户的功能都从这三个点添加。“Core\_xxx” 表示它用于驱动所有者。“App\_xxx” 表示它用于应用程序所有者。“Customer\_xxx” 表示它用于定制化。

### 5.2.1 命令注册到 cmdTable

我们现在有了不同阶段的入口点，我们希望每个开发人员都能遵循 MStar 的规则。我们不在这些入口点中使用函数调用，我们只在其中注册命令。例如：

我们将定制功能打包成不同的命令，然后通过函数 `Add_CommandTable` 注册这些命令。如果我们遵循此规则，我们可以获得一些优势：

1. 我们可以在 uboot 的控制台模式下通过命令测试定制行为。
2. 我们可以使用命令 `showtb` 来双重检查注册了哪些命令。这在出现失败设置时非常有用。

### 5.3 服务

现在，我们已经为定制化准备了一些命令和函数，例如 USB 升级、OAD 升级、网络升级等。我们可以使用这些基本服务来帮助客户开发其特定功能。

### 5.4 如何打包新命令

在 uboot 的环境中添加新命令非常简单。我们可以参考 `MsUtility.c` 和 `cmd_MsUtility.c`，可以从这些文件中找到许多示例。

### 5.5 如何在这些三个文件夹中添加一个新文件

1. 在特定文件夹中创建一个源文件。
    例如，如果要向 MStarCustomer 添加一个新文件，则需要在 `/MstarCustomer/src` 中创建一个新的 C 文件。
2. 编辑 `/MstarCustomer/makefile`
    将我们的新文件添加到 `COBJS` 中。
3. 如果有头文件，请将头文件放在 `/MstarCustomer/Include` 中。
