---
title: 在Ubuntu16.04上安装Cuda 8.0和 cuDNN 6.0
date: 2017-12-24 23:38:58
tags: 
 - ubuntu
 - cuda
 - cudnn
author: alfred.cai
---

安装rensorflow教程较多较杂，于是自己记录了一份。
由于现在Nvidia的cuda已经默认是9.0版本了，但是tensorflow还是只支持到cuda8和 cuDNN6。所以本教程也是根据tensorflow选择了老版本。

- 系统版本：Ubuntu-GNOME 16.04
- 显卡：Nviida 1070Ti

## 0安装 cuda8.0

很多教程都是先安装显卡驱动，再运行.run 文件的。在自己安装了几次之后个人还是喜欢下载.deb 文件，包管理器会自动安装显卡驱动的。

官网默认是cuda9，所以需要在[https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)找到8.0版本。
我们就点 cuda-80-ga2 ==> Linux ==> x86_64 ==> 16.04 ==> deb(local) 或者 deb(network)
下载“cuda-repo-ubuntu1604_8.0.61-1_amd64.deb”

```bash
sudo dpkg -i cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda-8-0
```

注意，其他教程里写的都是install cuda，但是我们不是安装最新的cuda，要选择成cuda-8-0

安装好之后建议重启一下，因为显卡驱动必须重启一下还能正常使用

### 配置cuda

我用的是zsh，所以是在.zshrc 里配置。默认的是bash，只需把.zshrc改成.bashrc就行了

```bash
nano .zshrc
```

在.zshrc的最前面写入环境变量的配置

```bash
export CUDA_HOME=/usr/local/cuda-8.0
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
export PATH=/usr/local/cuda-8.0/bin:$PATH
```

重新载入.zshrc

```bash
source .zshrc
```

### 测试cuda和显卡驱动

检查cuda的环境变量是否配置成功

```bash
nvcc --version
```

进入cuda代码样例(默认在用户文件夹下)检查显卡驱动是否安装成功

```bash
cd /home/*/NVIDIA_CUDA-8.0_Samples/1_Utilities/deviceQuery
make -j
./deviceQuery
```

## 1安装cuDNN6

安装cuDNN需要注册一个Nvidia的开发者帐号，当然也是免费的。网址[https://developer.nvidia.com/rdp/cuDNN-download](https://developer.nvidia.com/rdp/cuDNN-download)

注意要下载”Download cuDNN v6.0 (April 27, 2017), for CUDA 8.0“ ==> "cuDNN v6.0 Library for Linux"

```bash
tar -xzvf cuDNN-8.0-linux-x64-v6.0.tgz 
sudo cp include/cuDNN.h /usr/local/cuda/include/
sudo cp lib64/* /usr/local/cuda/lib64/
```

注意lib64里面其实只有两个文件，还有两个是软链接，复制后需要重新制作软链

```bash
cd /usr/local/cuda/lib64/
sudo rm -rf libcuDNN.so libcuDNN.so.6
sudo ln -s libcuDNN.so.6.0.21 libcuDNN.so.6
sudo ln -s libcuDNN.so.6 libcuDNN.so
```

## 2安装tensorflow

跟着官网去安装就好了，没有什么需要配置的内容了。

### 当然先建议将apt和pypi的安装源切换到国内源

个人常用的是清华的源[https://mirrors.tuna.tsinghua.edu.cn/help/pypi/](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/)

当然推荐一下母校的源，虽然这个是在我毕业之后才有的。。。[https://mirrors.shu.edu.cn/help/pypi.html](https://mirrors.shu.edu.cn/help/pypi.html)

### 安装方式纠结

会有人去纠结用virtualenv还是pip安装，个人的想法是如果一直都要用的可以用pip。如果是抱着试一试的心态，或是有处女座洁癖的码农可以用virtualenv，它可以创建出一个独立的python第三方库的环境。正式的一个项目或是网上下载的项目，建议还是用virtualenv，独立的环境能避免一些没必要的版本问题。
