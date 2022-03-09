[点击跳到android开发者页面](https://www.android-doc.com/tools/help/adb.html)

1.什么是adb
> adb 是android debug bridge的缩写，adb工具一般位于 <sdk>/platform-tools/目录下

2.adb的一些使用技巧

> 2.1 adb 查询模拟器/手机设备

```
命令: adb devices

输出:
    List of devices attached 
    emulator-5554  device
    emulator-5556  device
    emulator-5558  device
    
or
    no device

```
> 2.2 指定模拟器/手机执行命令

```
命令：adb -s <serialNumber> <command> 

输入exp：adb -s emulator-5556 install helloWorld.apk
```

> 2.3 安装apk

```
命令： adb install <path_to_apk>
```

> 2.4 从模拟器/手机 复制/拉取文件

```
命令：
    pull : adb pull <remote> <local>
    push : adb push <local> <remote>

输入exp：adb push foo.txt /sdcard/foo.txt
```

> 2.5 adb logcat

```
命令：[adb] logcat [<option>] ... [<filter-spec>] ...

输入：adb logcat
```


目录 | 命令 | 描述 | 备注
--- |---|---|---
Options | -d | 将adb命令直接指向唯一附加的USB设备。 | 如果附加了多个USB设备，则返回一个错误。
Options | -e| 将adb命令直接指向惟一运行的模拟器实例。 | 如果运行多个模拟器实例，则返回一个错误。
Options | -s <serialNumber>|直接adb命令一个特定的模拟器/设备实例，由它的adb指定的序列号引用，如emulator-5556 |如果没有指定，adb将生成错误。
---

目录 | 命令 | 描述 | 备注
--- |---|---|---
General | devices | 打印所有附加模拟器/设备实例的列表 | 
General | help| 打印受支持的adb命令列表 |
General | version |打印adb版本号 |

---

目录 | 命令 | 描述 | 备注
--- |---|---|---
Debug | logcat [<option>] [<filter-specs>] | 将日志数据打印到屏幕上 | 
Debug | bugreport | 将dumpsys、dumpstate和logcat数据打印到屏幕上，以便报告bug |
Debug | jdwp |打印给定设备上可用JDWP进程的列表 |您可以使用forward jdwp:<pid>端口转发规范来连接到特定的jdwp进程。例如:adb forward tcp:8000 jdwp:472 jdb -attach localhost:8000

---


目录 | 命令 | 描述 | 备注
--- |---|---|---
Data | install <path-to-apk> | 将Android应用程序(指定为.apk文件的完整路径)推送到模拟器/设备的数据文件。 | 
Data | pull <remote> <local> | 将指定的文件从模拟器/设备实例复制到电脑。 |
Data | push <local> <remote> |将指定的文件从开发计算机复制到模拟器/设备实例。 |

---

目录 | 命令 | 描述 | 备注
--- |---|---|---
Ports and Networking | forward <local> <remote> | 将Socket连接从模拟器/设备实例上的指定本地端口转发到指定远程端口。 | 
Ports and Networking | ppp <tty> [parm]...| 通过USB运行PPP，但不应该自动启动PPP连接。<tty> - PPP流的tty。例如dev: / dev / omap_csmi_ttyl。[parm]……-零或更多PPP/PPPD选项，如defaultroute、local、notty等。 |

---

目录 | 命令 | 描述 | 备注
--- |---|---|---
Shell | shell | 在目标模拟器/设备实例中启动远程shell。 | 有关更多信息，请参见发出Shell命令。
Shell | shell [<shellCommand>]| 在目标模拟器/设备实例中发出shell命令，然后退出远程shell。 |

---

目录 | 命令 | 描述 | 备注
--- |---|---|---
Server | start-server |检查adb服务器进程是否正在运行并启动它(如果没有)。 | 
Server | kill-server| 终止adb服务器进程。 |

---

目录 | 命令 | 描述 | 备注
--- |---|---|---
Scripting | get-serialno | 打印adb实例序列号字符串。 | 
Scripting | get-state| 打印模拟器/设备实例的adb状态。|
Scripting | wait-for-device |阻塞执行，直到设备联机为止——也就是说，直到实例状态为设备为止。 |
    
