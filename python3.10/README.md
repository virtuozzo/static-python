# Python 3.10.0 static compilation

### Prerequisites:
Ubuntu 18.04 VM with 4Gb RAM 2 VCPU
configuration stored in ```vm_dumpxml.xml``` file
uname -a output:
```Linux ubuntu18-buildserver 4.15.0-88-generic #88-Ubuntu SMP Tue Feb 11 20:11:34 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux```

## Build preparation process:
Install needed packages from file ```vm_installed_packages.txt```:
```
while read -r line; do apt-get -y install "$line"; done < vm_installed_packages.txt
```
Clone repo with packages and python:
```
git clone https://github.com/OnApp/static-python.git
```

## Phase I: install static compilator musl
```
export MUSL_PATH=/root/build/musl
mkdir $MUSL_PATH
tar -zxvf musl-1.2.2.tar.gz
cd musl-1.2.2/
./configure --prefix=$MUSL_PATH
make
make install
export CC=/root/build/musl/bin/musl-gcc
```
RESULT: ```$MUSL_PATH/bin/musl-gcc```

## Phase II: build zlib static lib
```
tar -xvf zlib-1.2.11.tar.xz
cd zlib-1.2.11/
CFLAGS="-static -fPIC" ./configure --static
make
```
RESULT: ```libz.a```

## Phase III: build OpenSSL static lib
```
mkdir /usr/local/ssl
tar -zxvf openssl-1.1.1l.tar.gz
CFLAGS="-static -idirafter /usr/include/ -idirafter /usr/include/x86_64-linux-gnu/" ./config no-shared --static --prefix=/usr/local/ssl1
```
This tip to include ```/usr/include/x86_64-linux-gnu/``` was got from https://github.com/openssl/openssl/issues/7207
```
make
make install
```
RESULT: ```/usr/local/ssl1/lib/libssl.a```
TIP: To choose required ssl version:

	Download python 3.10
	>>> import ssl
	>>> ssl.OPENSSL_VERSION

## Phase IV: Build Python static binary
```
tar -zxvf Python-3.10.0.tgz
mkdir build && cd build
../configure --with-static-libpython --enable-optimizations --with-ensurepip=no --prefix=/usr/pythoncontroller --disable-shared --disable-test-modules LDFLAGS="-static" CFLAGS="-static -idirafter $MUSL_PATH/include/ -s -Os -g0" CPPFLAGS="-static -idirafter $MUSL_PATH/include/ -s -Os -g0"
```
Provide actual Setup.local file after configure and before make. It could be copied from previous versions and edited.
```
cp ../../Setup.local Modules/
make build_all | tee install.log
make install
```
This builds certainly end with errors, it's ok. Please, check list of built packages.
RESULT: 
```/usr/pythoncontroller/bin/python```
```/usr/pythoncontroller/lib/libpython3.10.a```
```/usr/pythoncontroller/lib/python3.10/```
Now python should work correctly with this libs from /usr/pythoncontroller directory.

## Phase V: Prepare python to OnApp

Copy web related site-packages into lib folder from previous version of python build.
New version of web lib could be got from https://github.com/webpy/webpy
Remove unused libraries to optimize size:
```
config-3.10-x86_64-linux-gnu, pydoc_data, lib2to3, tkinter, turtledemo, zoneinfo, ensurepip
```
```/usr/pythoncontroller/lib/python3.10/lib-dynload``` should exist on target machine so we need to track it by git. Put ```.gitkeep``` file in that folder before push python into repositories. 
