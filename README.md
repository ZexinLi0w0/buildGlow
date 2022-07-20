# buildGlow
## Overview

Unofficial Guide to build [Glow](https://github.com/pytorch/glow) in Ubuntu 20.04

## Environment

Hardware: x64

Software: Ubuntu 20.10 groovy

## Dependencies preparation

Glow repo: https://github.com/pytorch/glow
```
git clone https://github.com/pytorch/glow.git
cd glow
git reset --hard f2efc9a77afec9681dbd4495e631c5bda72d5a8c # this version is tested succesfully.
git submodule update --init --recursive
```

fmt (official dependencies)
```
cd ~
git clone https://github.com/fmtlib/fmt
mkdir fmt/build
cd fmt/build
cmake ..
make
sudo make install
```

apt dependencies
```
sudo apt update
sudo apt upgrade # be careful
sudo apt-get install cmake graphviz libpng-dev libprotobuf-dev ninja-build protobuf-compiler \
    wget opencl-headers libgoogle-glog-dev libboost-all-dev \
    libdouble-conversion-dev libevent-dev libssl-dev libgflags-dev \
    libjemalloc-dev libpthread-stubs0-dev liblz4-dev libzstd-dev libbz2-dev \
    libsodium-dev libfmt-dev
sudo apt install scons
sudo apt-get install python-is-python3
```

boost (>1.51)
```
cd ~
wget https://boostorg.jfrog.io/artifactory/main/release/1.79.0/source/boost_1_79_0.zip
unzip boost_1_79_0.zip
cd boost_1_79_0/
sudo ./bootstrap.sh
sudo ./b2 install
```

!Deactivate anaconda and unset related environment variables (o.w. will cause runtime library conflict in configuration)
```
conda deactivate
mv $anaconda anaconda_bak
printenv | grep anaconda
unset $ENV_VARIABLES
```

overwrite thirdparty library `folly` to stable version (following [#5041](https://github.com/pytorch/glow/issues/5041))
```
cd glow/thirdparty/
rm -rf folly/
wget https://github.com/facebook/folly/archive/refs/tags/v2022.07.18.00.zip
unzip v2022.07.18.00.zip
cp -r folly-2022.07.18.00/ folly
```

double-conversion (without double-conversion will raise not-found-error in building process)
```
cd ~
git clone https://github.com/google/double-conversion.git
cd double-conversion/
sudo scons install
cmake .
make
sudo make install
```

jemalloc (without jemalloc will raise `cannot find -lJEMALLOC_LIB-NOTFOUND` in building process)
```
cd ~
wget https://github.com/jemalloc/jemalloc/releases/download/5.3.0/jemalloc-5.3.0.tar.bz2
tar -jxvf jemalloc-5.3.0.tar.bz2 jemalloc-5.3.0/
cd jemalloc-5.3.0/
./autogen.sh
./configure
make
sudo make install
```

LLVM-9 (following [#4211](https://github.com/pytorch/glow/issues/4211) [#4695](https://github.com/pytorch/glow/issues/4695) [#5979](https://github.com/pytorch/glow/issues/5979))
```
cd /usr/local/
sudo wget https://github.com/llvm/llvm-project/archive/refs/tags/llvmorg-9.0.1.zip
sudo unzip llvmorg-9.0.1.zip
cd llvm-project-llvmorg-9.0.1/
# be sure to add release flag o.w. memory may be exhausted
sudo cmake -S llvm -B build -G Ninja -DLLVM_ENABLE_RTTI=ON -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi" -DCMAKE_INSTALL_PREFIX=/usr/local/llvm-9/ -DCMAKE_BUILD_TYPE=Release
sudo cmake --build build
```

## Configuration && Build

Configuration
```
cd ~
# build in a clean directory
mkdir build_Debug/
cd build_Debug/
# gtest has some errors here, disable test via flags.
cmake -G Ninja ../glow -DLLVM_DIR=/usr/local/llvm-project-llvmorg-9.0.1/build/lib/cmake/llvm \ 
  -DCMAKE_C_COMPILER=/usr/local/llvm-project-llvmorg-9.0.1/build/bin/clang \ 
  -DCMAKE_CXX_COMPILER=/usr/local/llvm-project-llvmorg-9.0.1/build/bin/clang++ \ 
  -DCMAKE_BUILD_TYPE=Debug \
  -DGLOW_BUILD_TESTS=OFF
```

Build
```
ninja all -v
```

If everything goes right: should not see any error while building by ninja
```
ls ~/build_Debug/bin/
# should see:
# char-rnn fr2en include-bin lenet-loader model-compiler model-runner NodeGen png2bin resnet-runtime 
resnet-verify tracing-compare cifar10 image-classifier InstrGen mnist model-profiler model-tuner 
object-detector ptb resnet-training text-translator x-model-builder
```
