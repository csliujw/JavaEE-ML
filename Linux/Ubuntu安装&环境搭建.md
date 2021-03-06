# Linux系统的安装

## 获取镜像

用软通碟，UItraISO。

Ubuntu镜像去中科大镜像站下载

<a href="http://mirrors.ustc.edu.cn/">科大镜像</a>

- 获取安装镜像
- 选择安装发行版
- 选择发行版本

<img src="../pics/ubuntu/ubuntu_iso_download.png" style="float:left">

##  制作系统盘

- 找到软件 ultraiso，下载安装
- 下载ubuntu镜像
- 找到下载好的镜像，双击，默认会以ultraiso打开。
- 启动-->写入硬盘映像

<img src="../pics/ubuntu/ultraiso_1.png" style="float:left">

选择磁盘驱动器，其他的按照图里来就行，然后点击写入。

<img src="../pics/ubuntu/ultraiso_2.png" style="float:left">

----

# CUDA安装

##  大致流程

我先装的N卡驱动，再装的CUDA，然后CUDNN。

## 先卸载

如果您的机器有安装过相关的驱动和CUDA，那么可以使⽤以下命令进⾏彻底的删除：

```shell
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo apt-get autoclean
sudo rm -rf /usr/local/cuda*
```

##  显卡驱动的安装

如果是安装CUDA,那么这⼀步可以跳过的，因为在安装CUDA的时候会顺带安装显卡驱动，我是先安装显卡驱动再装CUDA的。

首先，禁⽤Nouveau驱动（有些是可以跳过这一步的）

```shell
sudo gedit /etc/modprobe.d/blacklist.conf
在最后两⾏添加：
blacklist nouveau
options nouveau modeset=0
sudo update-initramfs -u
sudo reboot
修改后需要重启系统。确认下Nouveau是已经被你干掉，使用命令： 
lsmod | grep nouveau
```

显卡驱动安装⽅式有三种：

- 使⽤ubuntu的软件仓库进⾏安装：打开软件与更新->附加驱动，选择对应的进⾏安装； 
- 使⽤PPA第三⽅仓库进⾏安装： sudo add-apt-repository ppa:graphics-drivers/ppa ， sudo apt update , 查看可⽤的显卡驱动 sudo ubuntu-drivers devices ，选择合适的进⾏安装： sudo apt install nvidia-xxx . 
- 去NVIDIA官⽹选择对应的进⾏安装：https://www.nvidia.cn/Download/index.aspx?lang=cn，可 以通过： lshw -numeric -C display 或者 lspci -vnn | grep VGA 查看显卡型号。

显卡安装后，重启电脑，然后用nvidia-smi查看是否安装成功。

如果执行`nvidia-smi`显示了显卡的信息说明安装成功。

<span style="color:green">**我个人采用的安装方式是**</span>

```shell
#  查看ubuntu合适的安装版本
ubuntu-drivers devices
```

用的虚拟机，所以无法显示合适的显卡驱动，如果是真实的物理机，会显示可用的驱动，安装一个就行。

<img src="../pics/ubuntu/nvidia_01.png" style="float:left">

CUDA版本的对应关系如下：

<img src="../pics/ubuntu/cuda_nvidia.png" style="float:left">

根据自己需要的cuda版本下载对应的驱动。

如：我需要CUDA 10.2.x 那么我的Driver Version 应该＞＝440.那么我可以安装  440 或 450

```shell
sudo apt install # 驱动名称，我安装的是推荐的版本 它给我推荐的是450
```

<span style="color:green">**然后重启电脑，再执行，一定要重启电脑！！**</span>

```shell
nvidia-smi # 查看驱动是否安装成功
```

## CUDA的安装

<a href="https://developer.nvidia.com/cuda-toolkit-archive">CUDA历史版本下载</a>

理论上runfile还是deb都是可以的，但是我deb安装不成功，所以用的runfile文件。

<img src="../pics/ubuntu/CUDA.png" style="float:left">

```shell
sudo sh cuda_10.0.130_410.48_linux.run
```

根据提⽰进⾏安装，如果安装了显卡驱动，这边就不要再选择安装驱动了。runfile,的安装形式可以多CUDA共存，推荐这种方式。

安装完CUDA都需要设置环境变量： 

```shell
安装完CUDA都需要设置环境变量：
打开.bashrc⽂件： sudo vim ~/.bashrc
在其末尾添加：
export CUDA_HOME=/usr/local/cuda # 这个是用来建立软链接的，通过这个进行多CUDA共存
export PATH=$PATH:$CUDA_HOME/bin 
export LD_LIBRARY_PATH=/usr/local/cuda10.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

# 生成软链接 /usr/local/cuda-10.0这个是我安装的cuda10.0的目录
sudo ln -s /usr/local/cuda-10.0 /usr/local/cuda
```

> **注意对应版本的更改，这个是cuda 10.0。**

更新环境变量： `source ~/.bashrc` 测试CUDA是否安装成功：`nvcc -V` 如果不成功，请按CUDA多版本共存的说明进行操作!

## CUDA 多版本共存

<a href="https://blog.csdn.net/sinat_36502563/article/details/102866033">参考博客</a>

实现多版本CUDA共存的第一步，就是要将先前添加到.bashrc里的环境变量路径全部指向cuda软链接，也就是环境变量的路径里所有cuda-x.0的名字都改成cuda，如下：

```shell
export CUDA_HOME=/usr/local/cuda 
export PATH=$PATH:$CUDA_HOME/bin 
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

原先安装的是CUDA-9.0，所以nvcc -V指令可以获得版本信息：

```shell
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Sep__1_21:08:03_CDT_2017
Cuda compilation tools, release 9.0, V9.0.176
```

安装新的CUDA-10.0时记得不要选择安装驱动且不要生成软链接，安装完成后可以在/usr/local/下看到cuda-9.0和cuda-10.0两个文件夹（cudnn的安装是把一个.h头文件和几个lib放到cuda的对应目录下面，记得sudo cp的时候写到真实的cuda-10.0这样的路径下，不要写到cuda软链接路径就好，这样不影响版本对应）。

将cuda-9.0切换成cuda-10.0的过程如下：

```shell
sudo rm -rf /usr/local/cuda  #删除之前生成的软链接
sudo ln -s /usr/local/cuda-10.0 /usr/local/cuda #生成新的软链接
```

完成之后即可看到nvcc -V输出的cuda版本已经变成了10.0。再想切回cuda-9.0只需要再用上述指令给9.0的路径生成新的同名软链接即可。

## CUDNN的安装

cudnn的安装非常简单

### 下载文件

按需求下载[cudnn的安装文件](https://developer.nvidia.com/rdp/cudnn-archive)：https://developer.nvidia.com/rdp/cudnn-archive

Tar File的下载如下图所示，选择红方框中的选项进行下载

<img src="../pics/ubuntu/CUDNN.jpg">

下载的是cudnn-*tgz的压缩包时，按下方指令进行安装：

首先解压缩下的cudnn压缩包文件

```shell
tar -xzvf cudnn-9.0-linux-x64-v7.tgz
```

执行安装，其实就是拷贝头文件和库文件并给予权限

```shell
sudo cp cuda/include/cudnn.h /usr/local/cuda/include

sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64

sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```

（2）验证安装是否成功

```shell
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

如果显示

```shell
#define CUDNN_MAJOR 7
#define CUDNN_MINOR 6
#define CUDNN_PATCHLEVEL 2
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)
```

则cudnn安装成功

----

#  PyTorch安装

按照官网的流程进行按照的话，是无法按照GPU版本的，即便选的是GPU的安装方式。所以，先通过官网的方式安装一下，看什么版本最合适，然后去清华镜像下载对应版本的GPU版本文件，离线安装。

或者直接取清华镜像找对应的GPU版本，离线安装。