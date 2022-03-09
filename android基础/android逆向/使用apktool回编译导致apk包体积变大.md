1.原因：使用了高版本（网上说是>2.0.2版本会有此问题）apktool工具进行反编译和回编译导致的，apktool.yml里面有两个属性

```
compressionType: false
doNotCompress:
...（此处为文件路径之类的）
如：
- arsc
- lua
- pto
- tbl
```
经排查，是 - lua路径导致apk体积变大
