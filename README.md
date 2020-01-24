# grpc-cross-compile

Steps for cross compiling whole of gRPC i686 as an example target.

## Attribution

- [StackOverflow Answer](https://stackoverflow.com/questions/52202453/cross-compiling-grpc-using-cmake)
- [RaspberryPi Example](https://github.com/grpc/grpc/blob/master/test/distrib/cpp/run_distrib_test_raspberry_pi.sh)

## Get toolchain for cross compilation
```sh
mkdir -p ~/cross/i686 && cd ~/cross/i686
curl -kO https://toolchains.bootlin.com/downloads/releases/toolchains/x86-i686/tarballs/x86-i686--glibc--stable-2018.11-1.tar.bz2
tar -jxf ./x86-i686--glibc--stable-2018.11-1.tar.bz2
```

## Get gRPC source, install golang and [gRPC dependencies](https://github.com/grpc/grpc/blob/master/BUILDING.md).
```sh
git -c http.sslVerify=false clone --recursive -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc ~/grpc
```

## Get gRPC source, install golang and [gRPC dependencies](https://github.com/grpc/grpc/blob/master/BUILDING.md).
```sh
git -c http.sslVerify=false clone --recursive -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc ~/grpc
# or cleanup if you already have gRPC source
cd ~/grpc && git reset --hard && git submodule foreach git reset --hard \
&& git reset --hard && git submodule foreach git reset --hard && rm -rf cmake/build
```

## Build gRPC for host environment
```sh
HOST=~/grpc/cmake/build/host

rm -rf ${HOST} && mkdir -p ${HOST} && cd ${HOST} \
&& cmake ../../.. -DBUILD_SHARED_LIBS=ON \
&& make
```

## Set various paths for target environment
```sh
toolchain=~/cross/i686/x86-i686--glibc--stable-2018.11-1
arch=i686
target=${toolchain}
bin=${target}/bin/${arch}-buildroot-linux-gnu
sysroot=${target}/${arch}-buildroot-linux-gnu/sysroot
TARGET_HOST=~/grpc/cmake/build/${arch}-buildroot-linux-gnu
```

## Build gRPC for target environment
```sh
rm -rf ${TARGET_HOST} && mkdir -p ${TARGET_HOST} && cd ${TARGET_HOST} \
&& cmake ../../.. -DCMAKE_CXX_COMPILER=${bin}-g++ \
            -DCMAKE_C_COMPILER=${bin}-gcc \
            -DCMAKE_AR=${bin}-ar \
            -DCMAKE_STRIP=${bin}-strip \
            -DCMAKE_LINKER=${bin}-ld \
            -DCMAKE_NM=${bin}-nm \
            -DCMAKE_OBJCOPY=${bin}-objcopy \
            -DCMAKE_OBJDUMP=${bin}-objdump \
            -DCMAKE_RANLIB=${bin}-ranlib \
            -DCMAKE_SYSROOT=${sysroot} -DCMAKE_CROSSCOMPILING=1 \
            -DCMAKE_FIND_ROOT_PATH_MODE_PROGRAM=NEVER \
            -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=ONLY \
            -DCMAKE_FIND_ROOT_PATH_MODE_INCLUDE=ONLY \
            -DCMAKE_FIND_ROOT_PATH_MODE_PACKAGE=ONLY  \
            -DRUN_HAVE_STD_REGEX=0 -DRUN_HAVE_POSIX_REGEX=0 \
            -DCMAKE_SYSTEM_PROCESSOR=${arch} \
            -DCMAKE_SYSTEM_NAME=Linux \
            -DBUILD_SHARED_LIBS=ON \
            -DgRPC_PROTOBUF_PACKAGE_TYPE=MODULE \
	    -D_gRPC_CPP_PLUGIN=${HOST}/grpc_cpp_plugin \
&& make
```
