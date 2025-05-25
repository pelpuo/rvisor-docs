# System Requirements
This section details how to set up R-Visor on a ubuntu machine. The following are the required tools:

* [QEMU](https://github.com/qemu/qemu)
* RISC-V Glibc toolchain
* RISC-V Newlib toolchain

## Setting Up QEMU
QEMU is an open-source emulator and virtualizer that allows you to run software for one hardware architecture on another. It is commonly used for system emulation, enabling developers to test, debug, and run software in a virtualized environment without needing physical hardware. 

Our setup uses QEMU to emulate a RISC-V architecture, in order to use R-Visor on other architectures such as x86 or AArch64.

1. First install the prerequisites
```bash
sudo apt update
sudo apt install libglib2.0-dev libpixman-1-dev ninja-build git make gcc
```
2. Pull the QEMU sources from github
```bash
git clone https://github.com/qemu/qemu
cd qemu 
git checkout v7.0.0
```
3. Build QEMU
```bash
./configure --target-list=riscv64-softmmu
make -j $(nproc --ignore=1) 
sudo make -j $(nproc --ignore=1) install 
cd ..
```

## Setting Up The RISC-V Glibc Toolchain

1. First install the prerequisites
```bash
sudo apt update 
sudo apt install gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu gawk texinfo libexpat1-dev 
```
2. Next get the sources from GitHub
```bash
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain.git 
cd riscv-gnu-toolchain 
```
3. Update the submodules 
```bash
git checkout rvv-0.8 
git submodule update 
```
4. Create the install directory
```bash
sudo mkdir -p /opt/riscv_imafd
sudo chown -R $(echo "$USER") /opt/riscv_imafd 
```
5. Build the Glibc toolchain for RISC-V
```bash
./configure --prefix=/opt/riscv_imafd --with-arch=rv64imafd --with-abi=lp64d 
sudo make linux -j $(nproc --ignore=1) 
```
6. Add the toolchain to path
```bash
export PATH=$PATH:/opt/riscv_ima/bin 
sed -i "$ a export PATH=\$PATH:/opt/riscv_imafd/bin" ~/.bashrc 
cd ..
```

## Setting Up The RISC-V Newlib Toolchain
To setup the RISC-V newlib toolchain, follow steps **1** to **4** from [Setting Up The RISC-V Glibc Toolchain](#setting-up-the-risc-v-glibc-toolchain)

Next, build the Newlib toolchain for RISC-V
```bash
./configure --prefix=/opt/riscv_newlib --with-arch=rv64imafd --with-abi=lp64d 
sudo make -j $(nproc --ignore=1) 
```



Additional information can be found on the [RISC-V GNU ToolChain's GitHub](https://github.com/riscv-collab/riscv-gnu-toolchain)
