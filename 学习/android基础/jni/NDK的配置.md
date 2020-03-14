1.编写native方法并生成.class文件

```
//编写native方法
 static {
        //指定引用的so库名称，一边编译生成的so库都是libxxx，我们只需要截取lib后面的xx即可，就是libffmpeg.so我们只需要ffmpeg
        System.loadLibrary("ffmpeg");
    }

//定义native方法
public static native String getFfmpegStr();
```

```
//生成.class文件
可以直接make project，也可以使用命令
javac 目标文件路径.java
```

2.生成.h文件

```
javah 包名+class文件类名
```

3.新建jni目录

```
创建jni文件目录，与main同级，然后复制上面的.h文件到jni目录下
```


4.编写.c文件

```
//引用jni头文件
#include "jni.h"
//生成的.h头文件
#include "com_blues_ffmpegdemo_FFmpeg.h"
JNIEXPORT jstring JNICALL Java_com_blues_ffmpegdemo_FFmpeg_getFfmpegStr
  (JNIEnv * env, jclass jobject){
  //具体逻辑
    return (*env)->NewStringUTF(env,"测试 jni");
  }

```


5.在jni目录创建 Android.mk文件

```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
#编译生成的文件的类库名字，与System.loadLibrary中引用的一致
LOCAL_MODULE    := ffmpeg
#要编译的c文件
LOCAL_SRC_FILES := ffmpeg.c
include $(BUILD_SHARED_LIBRARY)
```


6.在jni目录创建 Application.mk文件

```
//配置要编译的平台，all是编译所有
APP_ABI := armeabi-v7a arm64-v8a
//指定最小sdk版本，与gradle中配置的一致
APP_PLATFORM := android-23
```


7.下载并配置使用NDK

```
在gradle.properties下配置
android.useDeprecatedNdk=true

在local.properties配置ndk和sdk目录
```


8.配置abi filter

```
在defaultConfig配置
ndk{
    moduleName "ffmpeg"       //生成的so文件名字，调用C程序的代码中会用到该名字
    abiFilters "armeabi-v7a","arm64-v8a","armeabi"
    }
```


9.进行ndk-build
> 对jni目录执行 ndk-build，如果不在jni目录下执行ndk-build会报错

```
Android NDK: Could not find application project directory !    
Android NDK: Please define the NDK_PROJECT_PATH variable to point to it.   
```


10.配置sourceSet，指定so文件路径

```
sourceSets {
        main {
            //指定在app/libs目录，不指定可能会导致could not find "xxx.so"
            jniLibs.srcDirs = ['libs']
          
        }
    }
```


11.could not find "xxx.so"
> 检查手机cpu框架是否都通过ndk-build都生成了并在gradle中都配置了，其次检查指定so文件的路径。最好打个apk包出来查看是否apk包内存在对应框架的so文件。