# HOW TO Build MXNet from source

## Pre1: Install OpenBLAS from source

For this part, I mainly referred to [OpenBlas#Installation-from-source](https://github.com/xianyi/OpenBLAS#installation-from-source). For me, "Normal Compile" is enough.

**NOTE:** To avoid using sudo, install in my own directory.

## Pre2: Install OpenCV from source

I mainly referred to this article: [Installation in Linux](http://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html).

Also, a pdf version for this doc is provided [HERE](appendix/Installation_in_Linux-OpenCV_2.4.13.pdf)

Some tricks to boost installation:

1. Use zip instead of git repo.


**NOTE:** Avoid using `sudo`, install in my own directory.

## Pre3: Configure Environment Variables

Add the following snippet into `.bashrc` or `.bash_profile` in the home directory.

```sh
# Wen Ke's Environment for OpenBLAS, OpenCV and 

export OPENCV_HOME=$HOME/opencv-3.2.0
export OPENBLAS_HOME=$HOME/openblas-0.2.19

export PATH=$OPENBLAS_HOME/bin:$OPENCV_HOME/bin:$PATH
export C_INCLUDE_PATH=$OPENBLAS_HOME/include:$OPENCV_HOME/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=$OPENBLAS_HOME/include:$OPENCV_HOME/include:$CPLUS_INCLUDE_PATH
export LD_LIBRARY_PATH=$OPENBLAS_HOME/lib:$OPENCV_HOME/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=$OPENBLAS_HOME/lib:$OPENCV_HOME/lib:$LIBRARY_PATH
export PKG_CONFIG_PATH=$OPENCV_HOME/lib/pkgconfig:$PKG_CONFIG_PATH
```

## Uh...Finally: Build MXNet from source

Just follow the [official installation guide](http://mxnet.io/get_started/install.html).

There is also a [pdf format](appendix/Installing_MXNet_mxnet_documentation.pdf).

Major steps are:

1. Use `wget` to download and extract MXNet zip. Current stable release is [mxnet-0.10.0.zip](https://codeload.github.com/dmlc/mxnet/zip/v0.10.0). I don't recommend cloning the repo since it is too large.
2. Build MXNet.

```sh
cd mxnet
make -j $(nproc) USE_OPENCV=1 USE_BLAS=openblas
```

