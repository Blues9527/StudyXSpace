1.ndk和ffmpeg版本选择
> ndk尽量不要选择最新版本，建议r16b，ffmpeg建议4.0.5。失败例子，ndk r20b 与ffmpeg 3.3.9和ffmpeg 4.2.2。还有不要直接使用as下载的ndk，最好去官网下载

2.遇到的一些问题
- c comipler cannot create executables
> 降低ffmpeg版本，高版本ffmpeg使用clang进行编译，低版本使用gcc编译，可以通过‘whereis gcc’查看是否有安装gcc

- could not find xx.h
> 降低ndk版本，高版本的sysroot库与头文件分离

3./usr/local/ffmpeg/share/man/man1 : Permission denied
> 手动创建一下这个目录，然后手动授权一下

4.编译步骤
> 首先运行 ./configure看一下有没有报错，然后运行
```
./configure --prefix=/usr/local/ffmpeg --enable-gpl --enable-nonfree --enable-libfdk-aac --enable-libx264 --enable-libx265 --enable-filter=delogo --enable-debug --disable-optimizations --enable-libspeex --enable-videotoolbox --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --cc=clang --host-cflags= --host-ldflags= 
```
> 检查一下还有哪些库没有安装，都使用brew install xx去进行完安装，安装完毕后进行make一下，最好使用sudo make，make完没有报错后再使用 sudo make install 去安装ffmpeg，安装没有出错，再运行脚本去编译so文件

5.检查ffmpeg是否安装成功
> /usr/local/bin 目录下是否有ffmpeg ffplay ffprobe这三个文件，有就是成功了

6.build_android.sh 代码(借鉴的)

```
#!/bin/bash
ADDI_CFLAGS="-marm"
API=19
PLATFORM=arm-linux-androideabi
CPU=armv7-a
#自己本地的ndk路径。
NDK=/Users/lanhuajian/NDK/android-ndk-r16b
SYSROOT=$NDK/platforms/android-$API/arch-arm/
ISYSROOT=$NDK/sysroot
ASM=$ISYSROOT/usr/include/$PLATFORM
TOOLCHAIN=$NDK/toolchains/$PLATFORM-4.9/prebuilt/darwin-x86_64
#自己指定一个输出目录，用来放生成的文件的。
OUTPUT=/Users/lanhuajian/Desktop/ffmpeg/ffmpeg-4.0.5.tar/android_out
function build
{
./configure \
--prefix=$OUTPUT \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-avdevice \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=android \
--arch=arm \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-I$ASM -isysroot $ISYSROOT -Os -fpic -marm" \
--extra-ldflags="-marm" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make 
make install
}
build

```
