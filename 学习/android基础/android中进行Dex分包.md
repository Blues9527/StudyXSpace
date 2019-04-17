# 为什么要进行dex分包？
    
> Google中限定每个dex包里面最大的方法数是65535，当一个android项目总的方法数超过65535时，是无法打出apk包的，此时则应该进行dex分包。

# 如何进行dex分包？

> 首先，引入android-support-v4:multidex的jar包，或则在gradle里面开启  multiDexEnabled true，需要在 defaultConfig里面添加。

> 其次，在application里，要么继承MultidexApplication，要么在attachBaseContext()方法里进行multidex的注册，即调用 Multidex.install(this)方法。

> 接着，如果需要控制maindex里面的文件，则需要配置maindexlist(只是一个文件名，可自定义)，里面的内容可以将apk反编译后，写一个java小demo去遍历输出，注意maindexlist里面的内容，都是以 .class结尾的，且路径要用/分割而不是\。

> 最后,需要在gradle里面配置，如maindelist的路径，主dex里面最大的方法数等。

```
exmple：

  dexOptions {
        javaMaxHeapSize "4g"
        preDexLibraries = false
        additionalParameters = ['--multi-dex',
                                '--main-dex-list=' + project.rootDir.absolutePath + '/app/maindexlist.txt',//指定maindexlist的目录
                                '--minimal-main-dex',//maindex方法数最小化
                                '--set-max-idx-number=30000'//控制maindex里面的最大方法数
        ]
    }

```

## 注意事项

> 以上方法适用于2.3.3左右的 android plugin version，更高的版本如3.3.0就不适合了。

> 可以在 project structure 里面去修改 android plugin version

    
    

