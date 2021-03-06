# Compile intel SGX sdk on ubuntu20.04

## Apt

```bash
sudo apt install build-essential ocaml ocamlbuild automake autoconf libtool wget python libssl-dev git cmake perl libssl-dev libcurl4-openssl-dev protobuf-compiler libprotobuf-dev debhelper cmake reprepro unzip
```



## sdk install

Download from release page, don't use master branch, which is not stable

```bash
export DEB_BUILD_OPTIONS="nostrip"
./download_prebuilt.sh
sudo cp external/toolset/ubuntu20.04/{as,ld,ld.gold,objdump} /usr/local/bin
make sdk #DEBUG=1
make sdk_install_pkg #DEBUG=1
./linux/installer/bin/sgx_linux_x64_sdk_2.10.100.2.bin
```



我们需要安装在两个位置`/opt`和`/opt/intel`, 不要安装在本目录 . 输入填 `/opt` ,然后把 `/opt/sgxsdk` 和 `/opt/intel/sgxsdk` 软链接在一起

`DEBUG=1`  是optional的,一般不需要

## psw install
```bash
export DEB_BUILD_OPTIONS="nostrip"
make psw #DEBUG=1
make deb_psw_pkg #DEBUG=1
sudo make deb_local_repo
```

append to `/etc/apt/source.list`

```
deb [trusted=yes arch=amd64] file:/home/lyj/linux-sgx-sgx_2.10/linux/installer/deb/local_repo_tool/../sgx_debian_local_repo focal main
```

```bash
sudo apt update
sudo apt install libsgx-ae-epid libsgx-ae-le libsgx-ae-pce libsgx-ae-qe3 libsgx-ae-qve libsgx-aesm-ecdsa-plugin libsgx-aesm-epid-plugin libsgx-aesm-launch-plugin libsgx-aesm-pce-plugin libsgx-aesm-quote-ex-plugin libsgx-dcap-default-qpl-dev libsgx-dcap-default-qpl libsgx-dcap-ql-dev libsgx-dcap-ql libsgx-enclave-common-dbgsym libsgx-enclave-common-dev libsgx-enclave-common libsgx-epid-dev libsgx-epid libsgx-launch-dev libsgx-launch libsgx-pce-logic libsgx-qe3-logic libsgx-quote-ex-dev libsgx-quote-ex libsgx-uae-service libsgx-urts-dbgsym libsgx-urts sgx-aesm-service 
```

```bash
# optional
sudo apt install sgx-dcap-pccs
```



## build sgx-gdb

sgx-gdb依赖老版本的gdb. 

download gdb-9.1

```bash
wget https://ftp.gnu.org/gnu/gdb/gdb-9.1.tar.xz
extract gdb-9.1.tar.xz
cd gdb-9.1
mkdir build
cd build 
../configure --with-python=/usr/bin/python3 --prefix=/opt/gdb-9.1
make
sudo make install
```
modify `which sgx-gdb` , redirect GDB to /opt/gdb-9.1/bin/gdb

## test
```bash
cd ~/linux-sgx-sgx_2.10/SampleCode/SampleEnclave
make clean && make
sgx-gdb ./app

set debug-file-directory /usr/lib/debug
enable sgx_emmt
run
```

you should see when finished
```
  [Peak stack used]: 15 KB
  [Peak heap used]:  12 KB
  [Peak reserved memory used]:  0 KB

```

## After installation

```
sudo rm /usr/local/bin/{as,ld,ld.gold,objdump}
```

