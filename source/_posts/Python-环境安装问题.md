---
title: Python-环境安装问题
date: 2025-07-05 17:35:49
categories: [python]
---
### nvcc fatal : Unsupported gpu architecture 
问题在安装torch_scatter时出现(其他时候也可能，都是一个问题)

Error INFO:

    nvcc fatal : Unsupported gpu architecture 'compute_80'
nvcc版本过低
首先，查看nvcc版本

    nvcc --version

    //61 服务器输出
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2019 NVIDIA Corporation
    Built on Sun_Jul_28_19:07:16_PDT_2019
    Cuda compilation tools, release 10.1, V10.1.243
版本过低，只有10.1 ，正常需要11.*支持
之后查看本机安装的cuda

    ls -l /usr/local/ | grep cuda

    //同样61

    lrwxrwxrwx  1 root root   21 8月   2  2021 cuda -> /usr/local/cuda-11.0/
    drwxr-xr-x 17 root root 4096 8月   2  2021 cuda-11.0
    drwxr-xr-x 16 root root 4096 6月  22  2021 cuda-11.1

安装了cuda11但是nvcc指向的旧版本，现在需要将其指向新的版本(如果没安装11.*，安装之后修改)

    打开 .bashrc 或 .zshrc 文件：

    添加
    export CUDA_HOME=/usr/local/cuda-11.1
    export PATH=$CUDA_HOME/bin:$PATH
    export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH

    保存brash

    source ~/.bashrc

    再次确认版本

    nvcc --version

    //Show INFO:
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2020 NVIDIA Corporation
    Built on Tue_Sep_15_19:10:02_PDT_2020
    Cuda compilation tools, release 11.1, V11.1.74
    Build cuda_11.1.TC455_06.29069683_0


ok 解决

注意：nvcc 显示的和nvidia-smi 显示不是一个
### 7z-zstd-linux
论文实验用到一些无损的压缩算法，lzma这些，7z算是一个开源好用的(？疑惑)，问题在7z的linux并不支持zstd(facebook 压缩算法)这些算法

    4ED  303011B BCJ2
    EDF  3030103 BCJ
    EDF  3030205 PPC
    EDF  3030401 IA64
    EDF  3030501 ARM
    EDF  3030701 ARMT
    EDF  3030805 SPARC
    EDF        A ARM64
    EDF        B RISCV
    EDF    20302 Swap2
    EDF    20304 Swap4
    ED     40202 BZip2
    ED         0 Copy
    ED     40109 Deflate64
    ED     40108 Deflate
    EDF        3 Delta
    ED        21 LZMA2
    ED     30101 LZMA
    ED     30401 PPMD
     D     40301 Rar1
     D     40302 Rar2
     D     40303 Rar3
     D     40305 Rar5
    EDF  6F10701 7zAES
    EDF  6F00181 AES256CBC
可以看到需要的算法 $[ppmd,zstd,deflate,brotil,lzma]$ 中有几个并不支持，因此介绍以下项目[p7zip-project_p7zip-zstd](https://github.com/p7zip-project/p7zip)

使用master分支即可
#### p7zip-zstd install
    cd p7zip/CPP/7zip/Bundles/Alone2 && make -f makefile.gcc

之后顺利则会正常编译

但是！！有可能出现下面问题

    patchelf --force-rpath --set-rpath '$ORIGIN/' _o/lib/7z_addon_codec/libbrotlidec.so*
    /bin/sh: 1: patchelf: not found
    make: *** [../../7zip_gcc.mak:1366: _o/libbrotlienc.so] Error 127
原因在于项目使用patchelf来设置.so的文件的 rpath
    
    如果在自己电脑上使用，很简单
    sudo apt....


而若在服务器上，就很麻烦了 [patchelf](https://github.com/NixOS/patchelf/releases/),首先下载下来编译文件--linux

    # 创建本地工具目录
    mkdir -p $HOME/tools && cd $HOME/tools
    //当然，想放在哪里都可以，注意后面对应修改就可以了


    wget https://github.com/NixOS/patchelf/releases/download/0.18.0/patchelf-0.18.0-x86_64.tar.gz(可以直接下载的话)

    tar -xvzf patchelf-0.18.0-x86_64.tar.gz//下载的包，放在对应位置

    # 添加到 PATH
    export PATH="$HOME/tools/patchelf-0.18.0-x86_64:$PATH"
ok 之后再重新编译，结束之后 
     
    ./_o/bin/7zz i


    # output：

    Codecs:
    4ED  303011B BCJ2
    EDF  3030103 BCJ
    EDF  3030205 PPC
    EDF  3030401 IA64
    EDF  3030501 ARM
    EDF  3030701 ARMT
    EDF  3030805 SPARC
    EDF    20302 Swap2
    EDF    20304 Swap4
    ED     40202 BZip2
    ED         0 Copy
    ED     40109 Deflate64
    ED     40108 Deflate
    EDF        3 Delta
    ED        21 LZMA2
    ED     30101 LZMA
    ED     30401 PPMD
    ED   4F71101 ZSTD
    ED   4F71104 LZ4
    ED   4F71102 BROTLI
    ED   4F71106 LIZARD
    ED   4F71105 LZ5
    ED        21 FLZMA2
    ED   4F71001 LZHAM
    D     40301 Rar1
    D     40302 Rar2
    D     40303 Rar3
    D     40305 Rar5
    EDF  6F10701 7zAES
    EDF  6F00181 AES256CBC
ok，支持$[zstd,brotil]$了