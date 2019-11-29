# 如何在Linux中编译Mac OS X ClickHouse

Linux机器也可以编译运行在OS X系统的`clickhouse`二进制包，这可以用于在Linux上跑持续集成测试。如果要直接在Mac OS X上构建ClickHouse，请参考另外一篇指南： https://clickhouse.yandex/docs/zh/development/build_osx/

Mac OS X的交叉编译基于以下构建说明，请首先遵循它们。

# Install Clang-8

按照https://apt.llvm.org/中的说明进行Ubuntu或Debian安装。
例如，按照Bionic的命令如下：

```bash
sudo echo "deb [trusted=yes] http://apt.llvm.org/bionic/ llvm-toolchain-bionic-8 main" >> /etc/apt/sources.list
sudo apt-get install clang-8
```

# 安装交叉编译工具集

我们假设安装 `cctools` 在 ${CCTOOLS} 路径下

```bash
mkdir ${CCTOOLS}

git clone https://github.com/tpoechtrager/apple-libtapi.git
cd apple-libtapi
INSTALLPREFIX=${CCTOOLS} ./build.sh
./install.sh
cd ..

git clone https://github.com/tpoechtrager/cctools-port.git
cd cctools-port/cctools
./configure --prefix=${CCTOOLS} --with-libtapi=${CCTOOLS} --target=x86_64-apple-darwin
make install

cd ${CCTOOLS}
wget https://github.com/phracker/MacOSX-SDKs/releases/download/10.14-beta4/MacOSX10.14.sdk.tar.xz
tar xJf MacOSX10.14.sdk.tar.xz
```

# 编译 ClickHouse

```bash
cd ClickHouse
mkdir build-osx
CC=clang-8 CXX=clang++-8 cmake . -Bbuild-osx -DCMAKE_SYSTEM_NAME=Darwin \
    -DCMAKE_AR:FILEPATH=${CCTOOLS}/bin/x86_64-apple-darwin-ar \
    -DCMAKE_RANLIB:FILEPATH=${CCTOOLS}/bin/x86_64-apple-darwin-ranlib \
    -DLINKER_NAME=${CCTOOLS}/bin/x86_64-apple-darwin-ld \
    -DSDK_PATH=${CCTOOLS}/MacOSX10.14.sdk
ninja -C build-osx
```

生成的二进制文件将具有Mach-O可执行格式，并且不能在Linux上运行。