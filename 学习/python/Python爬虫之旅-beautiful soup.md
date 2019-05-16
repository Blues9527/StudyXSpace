beautiful soup通常配合解析器一起使用，如lxml

大致步骤：
> 通过requests去请求某个url，获取其返回的内容（html）

```
headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36 SE 2.X MetaSr 1.0"}

    response = requests.get(url, headers=headers)
```

> 然后通过Beautiful Soup的css选择器获取对应节点的内容

```
  soup = BeautifulSoup(response.text, 'lxml')
    # 使用css选择器获取class="article"的节点下面的所有li节点
    for index, li in enumerate(soup.select(".article li")):
        if (index < 10):
            print('歌曲排名：' + li.span.text)
            print('歌曲链接：' + li.a['href'])
            print('歌曲名：' + li.find(class_="icon-play").a.text)  # 使用方法选择器
            print('演唱者/播放次数：' + li.find(class_="intro").p.text.strip())
            print('上榜时间：' + li.find(class_="days").text.strip())
        else:
            print('歌曲排名：' + li.span.text)
            print('歌曲名：' + li.find(class_="icon-play").a.text)
            print('演唱者/播放次数：' + li.find(class_="intro").p.contents[2].strip())  # 方法选择器和节点选择器搭配使用
            print('上榜时间：' + li.find(class_="days").text.strip())

            print('—————————————————强力分隔符———————————————————')
```
