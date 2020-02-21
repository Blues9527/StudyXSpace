
### NDK配置
```
 ndk {
        //设置支持的SO库架构,一般手机都是前三个
        abiFilters 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'x86'
        //'x86_64', 'mips', 'mips64'
    }
```

### Dex配置
```
//multiDex的一些相关配置，这样配置可以让你的编译速度更快
        dexOptions {
            //最大堆内存
            javaMaxHeapSize "8g"
            //是否支持大工程模式
            jumboMode true
            //预编译
            preDexLibraries true
            //
            dexInProcess true
            //最大进程数
            maxProcessCount 8
            //线程数
            threadCount 8
        }
```

### 文件结构配置

```
 //配置目录
    sourceSets {
        main {
            //一般在出library中用的比较多
            //排除某个类文件
            java.exclude '**/xxx.java'
            //排除某个包下的所有类
            java.exclude '**/xx/**'

            //主要的文件配置
            manifest.srcFile 'src/main/AndroidManifest.xml'
            assets.srcDirs = ['src/main/assets']
            res.srcDirs = ['src/main/res']
            java.srcDirs = ['src/main/java']
            //次要的文件，如aidl jni等
            aidl.srcDirs=['xx/aidl']
            jniLibs.srcDir 'xx/jniLibs'
        }
    }
```

### 混淆配置
```
 buildTypes {
        release {
            //执行proguard混淆
            minifyEnabled true
            //Zipalign优化
            zipAlignEnabled true
            // 移除无用的resource文件
            shrinkResources false
            proguardFile getDefaultProguardFile('proguard-defaults.txt')
            //可以配置多个混淆配置文件
            proguardFile 'proguard-xxxx.pro'
            ...
            proguardFile 'proguard-xxxx.pro'
        }
        debug{
            //同上
            ....
        }
    }
```

### 签名配置
在android{}下配置
```
signingConfigs {
        release {
            storeFile file('文件相对路径')
            storePassword '签名密码'
            keyAlias '别名'
            keyPassword '别名密码'

            //开启v1签名
            v1SigningEnabled true
            //开启v2签名
            v2SigningEnabled true
        }

        debug{
           同上
        }
    }
    
    
 buildTypes {
        release {
            //签名
            signingConfig signingConfigs.release
        }
        debug{
             //签名
            signingConfig signingConfigs.debug
        }
    }
```

### 自定义aar/apk名称

```
//自定义aar
 android.libraryVariants.all {
                    variant ->
                        if (variant.buildType.name == 'release') {
                            variant.outputs.all {
                                outputFileName = "xxx_${releaseTime()}.aar"
                            }
                        }
                }
                
//自定义apk
android.applicationVariants.all {
                variant ->
                    if (variant.buildType.name == 'release') {
                        variant.outputs.all {
                            outputFileName = "xxx_${variant.buildType.name}_${releaseTime()}.apk"
                        }
                    }
            }
            
区别：aar是用的libraryVariants，apk用的是applicationVariants
```
