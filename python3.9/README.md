Python 3.8.1 static compilation
Prerquisites:
Ubuntu 18.04 VM with 4Gb RAM 2 VCPU
configuration stored in vm_dumpxml.xml file
uname -a output:
Linux ubuntu18-buildserver 4.15.0-88-generic #88-Ubuntu SMP Tue Feb 11 20:11:34 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

Build preparation process:
1. Install needed packages from file vm_installed_packages.txt
while read -r line; do sudo apt-get -y install "$line"; done < vm_installed_packages.txt

Phase I install static compilator musl:

2. Make mysl build directory where musl will be installed <musl_path>
mkdir /root/build/musl
3. Untar musl-1.1.12.tar.gz:
tar -zxvf musl-1.1.12.tar.gz
4. build and install musl =-gcc and libs at <musl_path>:
cd musl-1.1.12
./configure --prefix=<musl_path>
make && make install 

Phase II build zlib static lib
5. Untar zlib-1.2.11.tar.xz to <zlib_path>:
tar -zxvf zlib-1.2.11.tar.xz
6. configure and build zlib with musl:
CC='<musl_path>/bin/musl-gcc -static -fPIC' ./configure --static
make

Phase III build OpenSSL static lib
7. Make directory where openssl will be installed <openssl_path>:
mkdir /usr/local/ssl
8. Untar openssl-1.1.1d.tar.gz:
tar -zxvf openssl-1.1.1d.tar.gz
9. configure and build OpenSSL with musl install in <openssl_path>:
cd openssl-1.1.1d/
./config CC="${MUSL_PREFIX}"bin/musl-gcc no-shared -static --static -idirafter /usr/include/ -idirafter /usr/include/x86_64-linux-gnu/ --prefix=<openssl_path>
make -DOPENSSL_THREADS
make install -DOPENSSL_THREADS

Phase IV Build Python static binnary
10. Untar Python-3.8.1.tgz:
tar -zxvf Python-3.8.1.tgz
11. Edit Setup.local and add related paths to it sections(zlib.a....)
vi Setup.local
12. Copy configuration file with needed changes to Moudles dir:
cp Setup.local Python-3.8.1/Modules/
13. Configure and build python to <python_path>:
CC=<musl_path>/bin/musl-gcc ./configure --enable-optimization --with-ensurepip=no --prefix=<python_path> --disable-shared LDFLAGS="-static" --with-zlib=<zlib_path> CFLAGS="-static -idirafter <musl_path>/include/ -s -Os -g0" CPPFLAGS="-static -idirafter<musl_path>/include/ -s -Os -g0"

CC=<musl_path>/bin/musl-gcc make CFLAGS="-static -DOPENSSL_THREADS -idirafter <musl_path>/include/ -s -Os -g0" CPPFLAGS="-static -DOPENSSL_THREADS -idirafter <musl_path>/include/ -s -Os -g0"

CC=<musl_path>/bin/musl-gcc make install CFLAGS="-static  -idirafter <musl_path>/include/ -s -Os -g0" CPPFLAGS="-static -idirafter <musl_path>/include/ -s -Os -g0"

After all this steps you can find static python installation under <python_path> Its ooptimized by size.


 ./config CC="/root/static-python-master/python3.9/build/musl/bin/musl-gcc -static -idirafter /usr/include/ -idirafter /usr/include/x86_64-linux-gnu/" no-shared --static --prefix=/root/static-python-master/python3.9/build/openssl/ --release threads


CC="/root/static-python-master/python3.9/build/musl/bin/musl-gcc" ./configure --with-ensurepip=no --prefix=/root/static-python-master/python3.9/build/python/ --disable-shared LDFLAGS="-static" --with-zlib=/root/static-python-master/python3.9/build/zlib --with-openssl=/root/static-python-master/python3.9/build/openssl CFLAGS="-static -idirafter /root/static-python-master/python3.9/build/musl/include/ -s -Os -g0" CPPFLAGS="-static -idirafter /root/static-python-master/python3.9/build/musl/include/ -s -Os -g0"

CC="/root/static-python-master/python3.9/build/musl/bin/musl-gcc" make CFLAGS="-static -idirafter /root/static-python-master/python3.9/build/musl/include/ -s -Os -g0" CPPFLAGS="-static -idirafter /root/static-python-master/python3.9/build/musl/include/ -s -Os -g0"

CC=/root/static-python-master/python3.9/build/musl/bin/musl-gcc make install --prefix=/root/static-python-master/python3.9/build/python/ CFLAGS="-static -idirafter /root/static-python-master/python3.9/build/openssl/include/ -idirafter /root/static-python-master/python3.9/build/musl/include/ -s -Os -g0" CPPFLAGS="-static -idirafter /root/static-python-master/python3.9/build/openssl/include/  -idirafter /root/static-python-master/python3.9/build/musl/include/ -s -Os -g0"