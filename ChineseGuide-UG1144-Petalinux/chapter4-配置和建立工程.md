# 配置和构建
## 导入硬件配置

本节解释了使用新的硬件配置更新现有/新创建的 PetaLinux 项目的过程。这使您能够使 PetaLinux 工具软件平台为构建 Linux 系统做好准备，该系统可根据您的新硬件平台进行定制。

### 导入硬件工程的步骤
1. 进入到Petalinux的工程目录下面（假设你已经创建了一个Petalinux工程）
  
        cd <plnx-proj-root>

2. 使用 petalinux-config 命令导入硬件描述信息。需要提供一个包含有 .xsa 文件的路径。

        petalinux-config --get-hw-description <PATH-TO-XSA Directory>/<PATH-TO-XSA>

    *注：如果你在后续操作中修改指定目录下的 .xsa 文件。Petalinux会提示一条信息：*
    
        INFO: Seems like your hardware design:<PATH-TO_XSA Directory>/system.xsa has changed warning for all subsequent executions of the petalinux-config/petalinux-build commands.
    
    *此时需要再次执行上述命令来读取最新的 .xsa 文件。*

    执行petalinux-config后就会自动进入系统配置的顶层菜单。此外，在运行petalinux-config --get-hw-description 命令时，工具还会自动检测系统主要硬件变化。

    进入系统的配置菜单之后，确保 DTG Settings → (template) MACHINE_NAME 是选中的，并把template选择到对应的值上

    | BSP          | Machine                          |
    | ------------ | -------------------------------- |
    | ZCU102       | zcu102-rev1.0                    |
    | ZCU104       | zcu104-revc                      |
    | ZCU106       | zcu106-reva                      |
    | ZCU111       | zcu111-reva                      |
    | ZCU1275      | zcu1275-revb                     |
    | ZCU1285      | zcu1285-reva                     |
    | ZCU216       | zcu216-reva                      |
    | ZCU208       | zcu208-reva                      |
    | ZCU208-SDFEC | zcu208-reva                      |
    | ULTRA96      | avnet-ultra96-rev1               |
    | ZCU100       | zcu100-revc                      |
    | ZC702        | zc702                            |
    | ZC706        | zc706                            |
    | ZEDBOARD     | zedboard                         |
    | AC701        | ac701-full                       |
    | KC705        | kc705-full                       |
    | KCU105       | kcu105                           |
    | VCU118       | vcu118-rev2.0                    |
    | SP701        | sp701-rev1.0                     |
    | VCK190       | versal-vck190-reva-x-ebm-01-reva |
    | VMK180       | versal-vmk180-reva-x-ebm-01-reva |

    注：如果使用的是自定义的开发板，不要改变这个配置

    确保 Subsystem AUTO Hardware Settings 选项是被选中的，进入子选单后应该看到类似如下界面：
    
        Subsystem AUTO Hardware Settings
        System Processor (psu_cortexa53_0) --->
        Memory Settings --->
        Serial Settings --->
        Ethernet Settings --->
        Flash Settings --->
        SD/SDIO Settings --->
        RTC Settings --->

    
    Subsystem AUTO Hardware Settings 菜单允许大范围的修改硬件设置。
    
    此步骤可能需要几分钟才能完成，因为该工具解析了
    + \*更新设备树所需的硬件信息硬件描述文件
    + PetaLinux U-Boot 配置文件（仅用于Microblaze）
    + 基于 “Auto Config Settings --->” 和 “Subsystem AUTO Hardware Settings --->” 设置的内核配置文件 （仅用于Microblaze）


    --silentconfig 选项允许您重用先前的配置。旧配置在包含用于无人值守更新的指定组件的目录中具有文件名 CONFIG.old。

### Build系统镜像



1. 进入到Petalinux的工程目录下面：

        cd <plnx-proj-root>


2. 运行petalinux-build 来build整个工程
   
        petalinux-build

    
    此步骤生成设备树 DTB 文件、第一阶段引导加载程序（适用于 Zynq® 设备、Zynq® UltraScale+™ MPSoC 和 MicroBlaze™）、PLM（适用于 Versal™ ACAP）、PSM（适用于 Versal ACAP）和 ATF（适用于 Zynq UltraScale+ MPSoC 和 Versal ACAP）、U-Boot、Linux 内核、根文件系统映像。和引导脚本（MicroBlaze 不存在的 boot.scr）。最后，它生成必要的引导映像。

3. 编译过程会显示在终端控制台，请等待编译完成。     
    当编译完成之后，生成文件会被保存在\<plnx-proj-root>/images/linux 或者 /tftpboot 文件夹
    控制台显示编译进度。
    例如：
    
        $ petalinux-build

        [INFO] Sourcing buildtools
        [INFO] Building project
        [INFO] Sourcing build environment
        [INFO] Generating workspace directory
        INFO: bitbake petalinux-image-minimal
        NOTE: Started PRServer with DBfile: <plnx-proj-root>/xilinx-vck190-2021.1/
        build/cache/prserv.sqlite3, IP: 127.0.0.1, PORT: 43231, PID: 5719
        Loading cache: 100%
        |
        | ETA:
        --:--:--
        Loaded 0 entries from dependency cache.
        Parsing recipes: 100% |
        ############################################################################
        ############################################################| Time: 0:01:37
        Parsing of 3477 .bb files complete (0 cached, 3477 parsed). 5112 targets,
        243 skipped, 0 masked, 0 errors.
        NOTE: Resolving any missing task queue dependencies
        NOTE: Fetching uninative binary shim file:/<plnx-proj-root>/xilinxvck190-2021.1/components/yocto/downloads/uninative/
        5ec5a9276046e7eceeac749a18b175667384e1f445cd4526300a41404d985a5b/x86_64-
        nativesdklibc.tar.xz;sha256sum=5ec5a9276046e7eceeac749a18b175667384e1f445cd4526300a41
        404d985a5b (will check PREMIRRORS first)
        Initialising tasks: 100% |
        ############################################################################
        #########################################################| Time: 0:00:14
        Checking sstate mirror object availability: 100% |
        ############################################################################
        #################################| Time: 0:00:20
        Sstate summary: Wanted 1441 Found 1118 Missed 323 Current 0 (77% match, 0%
        complete)
        NOTE: Executing Tasks
        NOTE: Tasks Summary: Attempted 4363 tasks of which 3067 didn't need to be
        rerun and all succeeded.
        INFO: Failed to copy built images to tftp dir: /tftpboot
        [INFO] Successfully built project


### 默认Images

当您运行 petalinux-build 时，它会为 Versal™ ACAP、Zynq® UltraScale+™ MPSoC、Zynq-7000 设备和 MicroBlaze™ 平台生成 FIT 映像。还会生成 RAM 磁盘映像： rootfs.cpio.gz.u-boot。

完整的编译日志生成.log存储在 PetaLinux 项目的构建子目录中。
+ 最终的镜像：\<plnx-proj-root>/images/linux/image.ub就是FIT镜像。
+ 内核镜像（包括 RootFS）
  - Image：Zynq® UltraScale+™ MPSoC and Versal™ platform
  - zImage：Zynq-7000
  -  image.elf：MicroBlaze
+ 构建镜像位于\<plnx-proj-root>/images/linux  目录中。如果在 PetaLinux 项目的系统级配置中启用了该选项，则副本也将放置在 /tftpboot 目录中。



## 为 Versal ACAP 生成 Boot Image

本节介绍如何为 Versal ACAP 生成  BOOT.BIN


### Generate Boot Image

Boot Image 通常包括一个 PDI 文件（(imported from hardware design），PLM，PSM固件，Arm® trusted firmware, U-Boot, and DTB.


执行以下命令以.bin格式生成引导图像：

     petalinux-package --boot --u-boot

这条命令将会在 images/linux 文件夹中生成 BOOT.BIN, BOOT_bh.bin, 和 qemu_boot.img. 默认的DTB载入地址为0x1000. 有关更多信息，请参阅 Bootgen 用户指南 （UG1283）。

如果需要改变DTB的载入地址，使用下列命令：

    petalinux-package --boot --plm --psmfw --u-boot --dtb --load <load_address>


执行这条命令将会生成一个包含特定DTB地址的BOOT.BIN文件。你需要确保你同时在petalinux-config和U-Boot menuconfig中修改了对应地址。


注意： versal-qemu-multiarch-pmc.dtb 和 versal-qemu-multiarch-ps.dtb 是 QEMU 的 DTB，用来启动多架构 QEMU。



## 为  Zynq UltraScale+ MPSoC 生成 Boot Image

本节介绍如何为Zynq UltraScale+ MPSoC  生成  BOOT.BIN

### Generate Boot Image

引导镜像可以放入闪存卡或 SD 卡中。当您在板上供电时，它可以从引导镜像启动。 Zynq UltraScale+ MPSoC的引导镜像包括：first stage boot loader image, FPGA bitstream(optional), PMU firmware, ATF, and U-Boot.


执行以下命令以生成 .BIN 格式的镜像。

    petalinux-package --boot --u-boot --format BIN



