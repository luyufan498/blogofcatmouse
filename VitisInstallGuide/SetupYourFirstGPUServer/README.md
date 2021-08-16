# 【从零开始的Vitis 2021】手把手教你安装Vitis AI GPU服务器 
## 前言
如果你需要安装一个用于运行Vitis AI的GPU服务器，有以下几点需要注意：
* Vitis AI的代码CPU的多核需求并不是很高，相反CPU需要选择单核强劲的。实测发现很多程序是单线程的程序，只能用上一个核（很多处理图片的代码就是一个LOOP循环，完全没用多线程）。
* Vitis/Vivado/petalinux多核性能有要求，至少可以利用6-8核。
* 显卡30系貌似有兼容性问题（2021.8）, Docker能够正常启动，也能够识别到GPU，但是无法正确调用，似乎只能使用到显存，已知和tf版本以及cuda有关。
* 10系显卡成功/20系未测试
* 本教程不涉及Docker外环境安装，请参考 [官方教程](https://github.com/Xilinx/Vitis-AI/tree/master/tools/Vitis-AI-Quantizer) （本人配置未成功）。
* 内存请使用至少32G （推荐为64G）
* 硬盘空间根据具体情况来：Vitis AI Docker 需要至少 100GB左右， Vitis + Vivado 需要 200 GB ， 每个硬件工程 30 GB 左右，准备 1T SSD 可以保证相对充足的空间
* 推荐系统版本 ubuntu 16.04/18.04



## 安装硬件&安装系统
参考本人视频：https://www.bilibili.com/video/BV1UA411w7mA/
系统版本：(ubuntu 18.04)

## 基本环境配置
### 配置 SSH
参考文章 [ubuntu开启SSH服务远程登录](https://blog.csdn.net/jackghq/article/details/54974141)

1. 查看当前的ubuntu是否安装了ssh-server服务。默认只安装ssh-client服务。  
`dpkg -l | grep ssh`
2. 安装ssh-server服务  
`sudo apt-get install openssh-server`
3. 再次查看安装的服务, 如果看到 openssh-server的字样，表示SSH服务已经打开  
`dpkg -l | grep ssh`
4. 使用下面命令查看当前服务器的IP地址：
`ip addr show`
5. 获得IP地址后即可在windows中使用ssh进行连接

### 安装VIM （可选/推荐）

打开命令行输入：
`sudo apt install vim`


### 关闭自动更新 （可选/推荐）

自动更新有时候会在你需要手动更新文件的时候后台更新（会导致无法手动更新需要的包，必须等更新完才行），因此这里我选择关掉自动更新。

参考文章 [ubuntu 禁止/取消系统自动更新的方法](https://blog.csdn.net/abcwoabcwo/article/details/79658605)

打开文件：  
`sudo vim /etc/apt/apt.conf.d/10periodic`  

修改内容为：  
`  
APT::Periodic::Update-Package-Lists "0"; `   
`APT::Periodic::Download-Upgradeable-Packages "0";`    
`APT::Periodic::AutocleanInterval "0"; `  

手动更新列表（常用命令）：
`sudo apt update`

手动安装最新的更新：
`sudo apt upgarde`



## N卡/CUDA/CUDNN

为了使用 GPU Docker你需要准备一张显卡, 本人安装时使用了一张3090/实际进行量化测试时候使用的是1080 。

### 驱动 （可选/可以通过cuda安装）
[Ubuntu 18.04 安装 NVIDIA 显卡驱动](https://zhuanlan.zhihu.com/p/59618999)

推荐使用 Ubuntu 软件仓库中的稳定版本安装


1. 在终端输入：ubuntu-drivers devices 可以查看显卡的推荐驱动：
`ubuntu-drivers devices`

2. 输入下面命令：
`sudo ubuntu-drivers autoinstall`

3. 如果需要安装特定的驱动：
`sudo apt install nvidia-xxx`

### CUDA 安装
要是有由于CUDA本身会自带驱动，因此可以使用CUDA自带的驱动。 Cuda安装相当简单，参考官网即可：
这里选择ubuntu18.04的版本[Cuda toolkit 11.4](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=18.04&target_type=runfile_local)

1. 下载 Cuda 11.4 版本
`wget https://developer.download.nvidia.com/compute/cuda/11.4.1/local_installers/cuda_11.4.1_470.57.02_linux.run`

2. 安装
`sudo sh cuda_11.4.1_470.57.02_linux.run`

具体安装过程请参考视频：https://www.bilibili.com/video/BV1UA411w7mA/

**注**：如果提前安装了驱动，驱动安装包会有警告提示，可以忽略，并在选择安装内容时取消安装驱动。

3. 环境变量设置
安装完成后需要手动配置环境变量，具体参考安装完成后给出的提示信息。这里给出设置当前账号环境变量的方法

打开当前用户home目录下的 .bashrc，如果在 GUI 下操作需要可以使用 快捷键`CTRL + H` 显示隐藏文件。 

`vim ~/.bashrc`

添加下面两行：

`export PATH=/usr/local/cuda-11.4/bin${PATH:+:${PATH}}`  
`export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}`

4. 重新打开终端或者Source这个文件：  
`source ~/.bashrc`

5. 验证是否安装成功，可以通过下面命令（可选）：  
`NVCC --version`

### CUDNN的安装

这里使用的是离线安装方法

首先前往CUDNN网站下载相关安装包： https://developer.nvidia.com/rdp/cudnn-download （需要注册账号并且登录）


下载runtime包：  
`wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/8.2.2/11.4_07062021/Ubuntu18_04-x64/libcudnn8_8.2.2.26-1+cuda11.4_amd64.deb`

下载dev包：
`wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/8.2.2/11.4_07062021/Ubuntu18_04-x64/libcudnn8-dev_8.2.2.26-1+cuda11.4_amd64.deb`


安装runtime  环境
`sudo dpkg -i libcudnn8_8.2.2.26-1+cuda11.4_amd64.deb`

安装dev 环境
`sudo dpkg -i libcudnn8-dev_8.2.2.26-1+cuda11.4_amd64.deb`

环境安装需要按照顺序执行。

## Anaconda3

Anaconda的按照参考官网即可：  

首先安装依赖：  
`apt-get install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6`

下载安装包：
`wget https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh`


运行安装包：
`bash ./Anaconda3-2021.05-Linux-x86_64.sh`

## 安装Docker


卸载旧版  
` sudo apt-get remove docker docker-engine docker.io containerd runc`

安装依赖包  

`sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release`

添加 GPG key  
` curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`

设置稳定版仓库  
`echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`


安装docker  
`sudo apt-get update`  
`sudo apt-get install docker-ce docker-ce-cli containerd.io`


添加用户组  
`sudo groupadd docker`  
`sudo usermod -aG docker $USER`  
`newgrp docker `

测试  
 `docker run hello-world`



## Vitis AI GPU docker 
在安装GPU Docker之前，我们还需要安装 NVIDIA Docker Runtime https://github.com/luyufan498/Vitis-AI-ZH/blob/master/docs/quick-start/install/install_docker/README.md


### NVIDIA Docker Runtime
#### 安装

1. Install the repository for your distribution by following the instructions [here](http://nvidia.github.io/nvidia-container-runtime/).

```
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update
```

2. Install the `nvidia-container-runtime` package:
```
sudo apt-get install nvidia-container-runtime
```


#### Docker Engine 设置

1. Systemd drop-in file
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/override.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --host=fd:// --add-runtime=nvidia=/usr/bin/nvidia-container-runtime
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

2. Daemon configuration file
```bash
sudo tee /etc/docker/daemon.json <<EOF
{
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
EOF
sudo pkill -SIGHUP dockerd
```


### build Vitis AI GPU docker

下载Vitis AI
```
git clone --recurse-submodules https://github.com/Xilinx/Vitis-AI  
cd Vitis-AI
```

根据脚本本地生成GPU版本的镜像
```
cd setup/docker
./docker_build_gpu.sh
```

运行GPU版本
`./docker_run.sh xilinx/vitis-ai-gpu:latest`


