# 【从零开始的Vitis 2021】手把手教你安装GPU服务器
## 前言
如果你需要安装一个用于运行Vitis AI的GPU服务器，有以下几点需要注意：
* Vitis AI的代码CPU的多核需求并不是很高，相反CPU需要选择单核强劲的。实测发现很多程序是单线程的程序，只能用上一个核（很多处理图片的代码就是一个LOOP循环，完全没用多线程）。
* Vitis/Vivado/petalinux多核性能有要求，至少可以利用6-8核。
* 显卡30系貌似有兼容性问题（2021.8）, Docker能够正常启动，也能够识别到GPU，但是无法正确调用，似乎只能使用到显存，已知和tf版本以及cuda有关。
* 10系显卡成功/20系未测试
* 本教程不涉及Docker外环境安装，请参考 [官方教程](https://github.com/Xilinx/Vitis-AI/tree/master/tools/Vitis-AI-Quantizer) （本人配置未成功）。
* 内存请使用至少32G （推荐为64G）
* 硬盘空间根据具体情况来：Vitis AI Docker 需要至少 100GB左右， Vitis + Vivado 需要 200 GB ， 每个硬件工程 30 GB 左右，准备 1T SSD 可以保证相对充足的空间

## 安装硬件&安装系统
参考本人视频：https://www.bilibili.com/video/BV1UA411w7mA/
## 基本环境配置
### 配置 SSH
参考文章 [ubuntu开启SSH服务远程登录](https://blog.csdn.net/jackghq/article/details/54974141)

1. 查看当前的ubuntu是否安装了ssh-server服务。默认只安装ssh-client服务。  
`dpkg -l | grep ssh`
2. 安装ssh-server服务  
`sudo apt-get install openssh-server`
3. 再次查看安装的服务, 如果看到 openssh-server的字样，表示SSH服务已经打开
`dpkg -l | grep ssh`

## N卡/CUDA/CUDNN
## Anaconda
## Vitis AI GPU docker
