---
title: 在Windows环境下编译XGboost的R语言包GPU支持
mathjax: false
date: 2019-07-23 11:39:43
tags:
- machine-learning 
- xgboost 
- cuda 
- R-package
author: alfred.cai
---
由于XGboost的R包并没有对GPU的支持，而我又一定要用R语言，刚好官网上提供了自己编译能让R包支持CUDA。自己在学习实践XGboost时，想用他和随机森林的方法进行比较，需要构建大量的树，用上GPU希望能加快训练速度。但是编译的过程十分的痛苦，记录一下也许可以对别人有所帮助。  
<!-- more -->
## 环境介绍

*   Windows 10 1903
*   R 3.6.1
*   Rtools 35
*   Visual Studio 2019
*   CMake
*   cuda 10.1，这次编译使得xgboost能用上cuda 10.1版本

## 步骤
1.  下载代码到本地，包括它所用到的三个其他项目（rabit、dmlc-core、cub）

```bash
    git clone --recursive https://github.com/dmlc/xgboost  
    cd xgboost  
```

2.  用Visual Studio 2019打开项目，在他的CMake setting里的CMake command arguments里写入 `\-DUSE\_CUDA=ON -DR\_LIB=ON`
    *   如果你遇到了[LIBR\_LIB\_DIR was not set](#LIBR-LIB-DIR) 或是[INTERFACE\_INCLUDE\_DIRECTORIES property contains path](#INTERFACE-INCLUDE-DIRECTORIES) 问题，你需要多加一些参数，最后的命令会是：`\-DUSE\_CUDA=ON -DR\_LIB=ON -DLIBR\_EXECUTABLE="C:/Program Files/R/R-3.6.1/bin/x64/R.exe" -DCMAKE\_INSTALL\_PREFIX="C:/Program Files (x86)/xgboost"`  
    *   Visual Studio 里的Build Root与官方文档不一致，我就选择了它默认的`${projectDir}\out\build\${name}`
        
    *   如果你用命令行下的cmake的话，cuda 10.1 版本的机器会提示没有找到nvcc，所以我自己尝试下来只能在Visual Studio图形化界面下操作

3.  选择`Build > Build ALL`进行编译，等待编译完成后再点击`Build > install xgboost`它会自动帮你安装R包，结束后就可以打开RStudio进行测试了。
    
    *   或者你也可以在开始菜单里找到Visual Studio下的命令行，执行`cmake --build . --target install --config Release`进行编译，编译完成后它会自动安装R包。结束后可以打开RStudio进行测试。两者做的同一件事，只是操作不一样，看个人喜好吧。

4.  python版的安装，需要回到主目录下的`python-package`进行安装，安装完后可以用自带的测试程序进行测试是否安装成功

```bash
cd python-package  
python setup.py install  
python ..\\tests\\benchmark\\benchmark\_tree.py --tree\_method "gpu\_hist" --iterations 10  
python ..\\tests\\benchmark\\benchmark\_tree.py --tree\_method "hist" --iterations 10  
```

## 可能遇到的问题

由于我的CMake是用Visual Studio下载的，是内嵌在里面的，用起来不是很方便但是还可以用用。如果想用cmake的命令行，可以在开始菜单里找到Visual Studio 2019的文件夹下X64 Native Tools Cammand Prompt for VS 2019.

### LIBR_LIB_DIR

> CMake Error at cmake/modules/FindLibR.cmake:44 (message): LIBR\_LIB\_DIR was not set!

这是因为R语言没有添加到系统环境里，可以在执行cmake的时候指定R语言的目录`-DLIBR_EXECUTABLE="C:/Program Files/R/R-3.6.1/bin/x64/R.exe"`

### INTERFACE_INCLUDE_DIRECTORIES

> CMake Error in C:\\Users\\alfre\\Documents\\CodeProjects\\GithubProjects\\xgboost\\CMakeLists.txt:  
> Target “xgboost” INTERFACE\_INCLUDE\_DIRECTORIES property contains path:  
> “C:/Users/alfred/Documents/CodeProjects/GithubProjects/xgboost/out/install/x64-Debug/include”  
> which is prefixed in the source directory.

这个报错是在Visual Studio生成配置文件时出现的，对比了用命令行生成的文件，发现是z指定到了`"C:/Program Files (x86)/xgboost"`，如果遇到的朋友可以在参数里再加一条`-DCMAKE_INSTALL_PREFIX="C:/Program Files (x86)/xgboost"`， 编译结束之后cmake也没生成`C:/Program Files (x86)/xgboost`文件夹。

### gendef file or dlltool file

这问题是由于没有没有安装Rtools导致的，奇怪的是，安装程序它会把`C:\Rtools\bin`添加到系统环境里，但是xgboost用到的两个程序其实是在Rtools所提供的mingw里，也就是说你要自己将`C:\Rtools\mingw_64\bin`添加到系统环境里的PATH变量里。我一开始以为我安装时取消勾选里这个选项，但是没有添加到环境变量里，但是后来又重装了一遍才发现其实是Rtools自己的问题。

### cuda版本问题

cuda版本问题永远在你身边！每次拿到新电脑，都是先装cuda再去安装TensorFlow，numpy等等软件包的，你会发现大家的cuda版本都是nvidia出的慢半个版本，导致你会在自己编译还是重装cuda中纠结。而且各家活跃度不一样使得对cuda支持的版本都不一样，所以在问题会不停的出现纠结花好久才解决。  
这次也是，如果根据官网的文档，在命令行下运行cmake（`cmake .. -G"Visual Studio 16 2019" -DUSE_CUDA=ON -DR_LIB=ON -DLIBR_EXECUTABLE="C:/Program Files/R/R-3.6.1/bin/x64/R.exe"`）会出现报错，发现官方支持到cuda10.0。但是！如果用Visual Studio打开项目，用Visual Studio去生成缓存文件，发现并不会报错，能让xgboost支持cuda10.1。最近新买电脑的朋友的可以注意下了，装cuda10.1也没问题。