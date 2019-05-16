1.选用的库 selenium

大致步骤：

```
 self.browser = webdriver.Chrome('E:\chromedriver_win32\chromedriver')
 self.browser.get('https://www.pearvideo.com/')
```

> 首先初始化类对象的时候，启动webdriver.Chrome();并且输入想要爬取的网址

> 接着self.browser.find_elements_by_class_name('actcontbd')找到对应的类名标签内容，注意是elements而不是element，否则会报错

> 然后就是根据上面找到的标签内容逐步去获取自己想要的内容，如：

```
video_url = video_info.find_element_by_xpath("//*[@href]")//获取url

video_title = video_info.find_element_by_class_name('actcont-title')//获取标题

video_cover = video_info.find_element_by_class_name('img')//获取封面
```

> 使用完之后需要调用quit()方法去退出
