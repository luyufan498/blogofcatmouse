# 第三章：创建一个工程
## 使用BSP创建工程

PetaLinux 板级支持包 （BSP） 是Xilinx官方支持的开发板上的参考设计。你可以根据这个参考设计来定制化和修改你自己的工程，也可以以此为基础，创建你自己的官方开发板工程。PetaLinux 板级支持包是通过 “可安装的BSP” 文件的形式提供 ***（注：更像压缩包）***，BSP中包括所有必要的设计和配置文件、预建和测试的硬件以及软件图像，可准备在您的板上下载或在 QEMU 系统模拟环境中启动。 Petalinux本身不会包含任何BSP包，你需要单独的下载和安装。你可以在 [Xilinx 下载中心](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html) 下载到相关BSP。此外，每个BSP中还好有一个README来解释BSP的详细信息。


### 准备开始

在运行本章节的教程之前，你需要满足以下条件：
+ 提前下载好BSP文件
+ 设置好Petalinux的开发环境


### 从BSP创建工程

更改到要创建 PetaLinux 项目的目录。例如，如果您想在 /home/user 目录下创建的你工程：

    cd /home/user

命令控制台上运行petalinux-create命令：

    petalinux-create -t project -s <path-to-bsp>

*注：\<path-to-bsp> 替换成BSP的路径目录*

随后，Petalinu根据下载的BSP文件安装，同时你应该在控制台看到下列输出：

    INFO: Create project:
    INFO: Projects:
    INFO: * xilinx-zcu102-v<petalinux-version>
    INFO: has been successfully installed to /home/user/
    INFO: New project successfully created in /home/user/

*注意：PetaLinux 至少需要 50 GB 和最多 100 GB “/tmp" 空间才能成功构建项目。*

*注意：/tmp不可以在NFS系统上，具体参考 [Petalinx2021](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2021_1/ug1144-petalinux-tools-reference-guide.pdf) 第19页。手动指定缓存路径可以通过以下命令执行，需要注意缓存目录不可以被多个工程共享。petalinux-create -t project -s \<PATH_TO_PETALINUX_PROJECT_BSP> --tmpdir \<TMPDIR PATH>*




### 通过Vivado设计套件配置硬件平台

您可以使用 Vivado ®工具创建自己的硬件平台。无论硬件平台是如何创建和配置的，都需要进行少量的硬件 IP 和软件平台配置更改，以便让硬件平台 Linux 做好准备。

#### Zynq UltraScale+ MPSoC and Versal ACAP

为了启动Linux，Zynq® UltraScale+™ MPSoC 和  Versal™ ACAP 的硬件工程需要满足以下要求。

+ 至少64MB的外部内存空间 （必须）。
+ UART接口用于串口通讯 （必须）。
+ 非易失性存储，例如QSPI flash 和 SD/MMC。This memory is optional but only JTAG boot can work.
+ 网口（可选，网络连接的必要配置）

*注意：如果使用带中断的软 IP 或带中断的外部 PHY 设备，确保中断信号也连接到PS。*


#### Zynq-7000 系列设备

+ Triple Timer Counter （TTC） （必须）。注意，如果启用多个 TTC，Zynq-7000 Linux 内核将使用设备树上的第一个 TTC 块。请确保这个 TTC 没有被其他模块使用。
+ 具有至少 32 MB 内存的外部内存控制器（必须）。
+ UART （必须）
+ 非易失性存储，例如QSPI flash 和 SD/MMC。This memory is optional but only JTAG boot can work.
+ 网口（可选，网络连接的必要配置）


*注意：如果使用带中断的软 IP 或带中断的外部 PHY 设备，确保中断信号也连接到PS。*

#### MicroBlaze 处理器 (AXI)
+ 必要的IP核：
  + 具有至少 32 MB 内存的外部内存控制器（必须）。
  + 带有中断的双通道计数器（Dual channel timer）（必须）
  + 带有中断的 UART（必须）
  + 非易失性存储 例如 Linear or SPI 闪存 （必须）
  + 带中断的网口 （可选）

+ MicroBlaze 处理器配置：
  + MicroBlaze processors with MMU support by selecting either Linux with MMU or low-end Linux with MMU configuration template in the MicroBlaze configuration wizard.  
  *注：除非您了解此类更改的影响，否则不要禁用模板启用的任何指令设置的相关选项。*
  
  + 如果系统从非易失性存储启动，MicroBlaze 初始化boot loader的时候需要至少4KB的RAM给parallel flash使用 （寻址？）以及8KB的RAM给SPI flash使用。

*注：Petalinux 只支持32位 MicroBlaze 处理器*


### 把硬件平台导入到Petalinux的工程中

假设您已经完成了在Vivado中的设计工作，我们就可以把Vivado中的工程导出给Petalinux来创建系统了。

#### 导出硬件平台/设计

在配置硬件项目后，PetaLinux 项目需要一个硬件描述文件 （.xsa 文件）来提供有关处理系统的信息。您可以通过从 Vivado 设计套件中运行" Export Hardware"来获取硬件描述文件®。

在项目初始化（或更新）期间，PetaLinux 生成设备树源文件、U-Boot 配置头文件（仅限MicroBlaze处理器），并基于硬件描述文件启用 Linux 内核驱动程序（仅限MicroBlaze处理器）。具体细节见UG1144附录 B 。

对于Zynq® UltraScale+™ MPSoC平台，您需要通过 *Platform Management Unit(PMU) firmware和 ATF* 来启动。**见附录C**。如果您想要为 Cortex ®-R5F 构建 first stage boot loader（FSBL），则必须使用 Vitis ™软件平台构建它，因为使用 PetaLinux 工具构建的 FSBL 只能用于 Cortex-A53 启动。有关如何使用 Vitis 软件平台为 Cortex-R5F 构建 FSBL 的详细信息，参考 UG1137。

对于Versal平台，启动需要 PLM, PSM 和 ATF。**见附录 C**。


### 从模板创建一个新的工程

#### 创建一个新的工程

petalinux-create 命令可以用于创建新的工程：

     petalinux-create --type project --template <PLATFORM> --name <PROJECT_NAME>

参数的说明如下：
+ --template \<PLATFORM> 支持以下参数：
  + versal （Versal ACAP）
  + zynqMP （Zynq UltraScale+ MPSoC）
  + zynq （zynq7000系列）
  + microblaze
+ --name \<PROJECT_NAME> 工程的名字


