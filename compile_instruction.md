# **Instructions for building PaddlePaddle CPU from Source**

Author: Shubo Wang | <shubo.wang@globusmedical.com>

Note: Adapted from PaddlePaddle official document

This instruction is only for building CPU version. Please refer to paddlepaddle homepage for GPU version.

## **Compile For Windows 10**

Create your python environment for compilation. This python env should have the same python version as your build target. If want to build for python3.9, use python3.9 env. Virtualenv is recommended by paddle official while conda env has been tested by Shubo Wang. It is left for the user to chooce.

Activate your python environment and install necessary modules.

```
# assuming python env has been activated
pip install numpy, protobuf, wheel
pip install ninja
```

### A. Compile with Ninja (Recommanded)

```
mkdir build && cd build
cmake .. -GNinja -DWITH_GPU=OFF
ninja install all
```

### B. Compile with cmake and MSVS 2017

```
cmake .. -GNinja -G "Visual Studio 15 2017" -A x64 host=x64 -DWITH_GPU=OFF
```

Open the solution file in build folder and build all for x64 release

## **Compile for Linux**

## *Option 1: Docker*

### 1. Pull PaddlePaddle docker image for cpu

```
docker pull registry.baidubce.com/paddlepaddle/paddle:latest-dev
```

The docker image has python 3.7, 3.8, 3.9 and 3.10 installed. *Choose proper python interpreter for your build at step 3*

### 2. Create and enter a docker container

```
docker run --name paddle-test -v $PWD:/paddle --network=host -it registry.baidubce.com/paddlepaddle/paddle:latest-dev /bin/bash
```

- --name paddle-test: names the Docker container you created as paddle-test;
- -v $PWD:/paddle: mount the current directory to the /paddle directory in the docker container (PWD variable in Linux will be expanded to absolute path of the current path);
- -it: keeps interaction with the host;
- registry.baidubce.com/paddlepaddle/paddle:latest-dev: use the image named registry.baidubce.com/paddlepaddle/paddle:latest-dev to create Docker container, /bin/bash start the /bin/bash command after entering the container.

### 3. Prepare for build

```
Pip3.9 install protobuf
apt install patchelf

# checkout a branch for a specific version, for example release/2.4.2.1
cd paddle
git checkout NAME_BRANCH

mkdir -p /paddle/build && cd /paddle/build
cd build
```

### 4. Build

```
cmake .. -DPY_VERSION=3.9 -DWITH_GPU=OFF
make -j$(nproc)
```

## *Option 2: DIY (Not suggested)*

### Environment Requirment

- CentOS 6 (not recommended, no official support for compilation problems)
- CentOS 7 (GPU version supports CUDA 10.1/10.2/11.1/11.2/11.6/11.7)
- Ubuntu 14.04 (not recommended, no official support for compilation problems)
- Ubuntu 16.04 (GPU version supports CUDA 10.1/10.2/11.1/11.2/11.6/11.7)
- Ubuntu 18.04 (GPU version supports CUDA 10.1/10.2/11.1/11.2/11.6/11.7)

Compilation on linux requires gcc 8.2.0 which has to be built from source. Default gcc-8 version is 8.4.0 through apt-get, but it doesn't work.

### 1. Setup Env

Activate your python environment; could use virtualenv or conda

```
sudo apt update -y
sudo apt install -y bzip2 make
sudo apt install -y patchelf
mkdir build && cd build
```

```
# setup env variable
export PYTHON_INCLUDE_DIR=$(python -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())")
export PYTHON_LIBRARY=$(python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())")
export PYTHON_VERSION=$(python -c "from distutils.sysconfig import get_python_version; print(get_python_version())")
```

### 2. Build

```
# configure the project
cmake .. -D CMAKE_C_COMPILER=/usr/local/gcc-8.2/bin/gcc \
    -D CMAKE_CXX_COMPILER=/usr/local/gcc-8.2/bin/g++ \
    -DPY_VERSION=${PYTHON_VERSION} \
    -DPYTHON_INCLUDE=${PYTHON_INCLUDE_DIR} \
    -DPYTHON_LIBRARY=${PYTHON_LIBRARY} -DWITH_GPU=OFF 

# build
make -j$(nproc)
```
