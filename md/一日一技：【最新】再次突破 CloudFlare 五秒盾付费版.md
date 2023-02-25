> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wa0sdimbB5QEZKquDQsotw)

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqflibEJmgS2kHUXSxLR6zWvevQFXlYVUb6jkYNHRXbJMX46gJbwsPwfJfEEhwk3qtVGsNCLlTqM21w/640?wx_fmt=png)

摄影：产品经理

卤水金钱肚  

去年我写了一篇文章：[一日一技：如何捅穿 Cloud Flare 的 5 秒盾](https://mp.weixin.qq.com/s?__biz=MzI2MzEwNTY3OQ==&mid=2648980828&idx=1&sn=0b053e7284442c8e6beb073a61ec8b29&scene=21#wechat_redirect) ，这篇文章使用的第三方库『cloudscraper』可以绕过免费版的五秒盾。但遇到付费版就无能为力了。

最近在爬币圈的网站，其中有一个网站叫做：Codebase[1] 使用的就是付费版的 CloudFlare 五秒盾。当我们使用 CloudScraper 去爬时，报错如下：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqflibEJmgS2kHUXSxLR6zWvehbKkiabM4fFBiazwCPro3K0FwSRr6Pq71zPyykJCqm6EAdj1ibgX7GMbQ/640?wx_fmt=png)

那么现阶段，付费版的 CloudFlare 五秒盾，有没有什么办法绕过呢？其实方法非常简单。只需要使用 Docker 运行一个容器就可以了。启动命令为：

```
docker run -d \
  --name=flaresolverr \
  -p 8191:8191 \
  -e LOG_LEVEL=info \
  --restart unless-stopped \
  ghcr.io/flaresolverr/flaresolverr:latest


```

这个容器启动以后，会开启 8191 端口。我们通过往这个端口发送 http 请求，让他转发请求给目标网站，就可以绕过五秒盾。

具体使用示例：

```
import requests
import json

url = "http://localhost:8191/v1"

payload = json.dumps({
  "cmd": "request.get",
  "url": "https://www.coinbase.com/ventures/content",
  "maxTimeout": 60000
})
headers = {
  'Content-Type': 'application/json'
}

response = requests.post(url, headers=headers, data=payload)

# 这个Docker镜像启动的接口，返回的数据是JOSN，网页源代码在其中的.solution.response中
print(response.json()['solution']['response'])


```

访问效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqflibEJmgS2kHUXSxLR6zWveMrXeySF2LKibtpouVshyhC8zqcetE6YSp1gNLhmbZKo6sbynZY0gUbg/640?wx_fmt=png)

我们再写几行代码来提取一下标题：

![](https://mmbiz.qpic.cn/mmbiz_png/ohoo1dCmvqflibEJmgS2kHUXSxLR6zWveYUzI0OwCmOibj0ticH9v3AU6DUucgO8HTADeicDLmcuibuMibqqVkbAF4MQ/640?wx_fmt=png)

我们启动的这个容器，为什么可以绕过 CloudFlare 的五秒盾呢，关键原因就在这个项目中：FlareSolverr[2]。大家可以阅读他的源代码，看看他是怎么绕过的。

[1]

Codebase: _https://www.coinbase.com/ventures/content_

[2]

FlareSolverr: _https://github.com/FlareSolverr/FlareSolverr_

END

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/ohoo1dCmvqcLS5eIyiaZwBibFZxQ0rY6Cxqniaud08QzpsQje8n4junXCTJ0Z7weSCiaDr9DnaYrPqLEMIm1nMxfdQ/640?wx_fmt=jpeg)

未闻 Code· 知识星球开放啦！

一对一答疑爬虫相关问题

职业生涯咨询

面试经验分享

每周直播分享

......

未闻 Code· 知识星球期待与你相见~

![图片](https://mmbiz.qpic.cn/mmbiz_gif/ohoo1dCmvqdibWI839rMdKvybARQ46RLEMxasjDASe6tgoJDOk7RibADqoqykrOX4uux8euic2wYGcp6iajaVyCzsQ/640?wx_fmt=gif)  

一二线大厂在职员工

十多年码龄的编程老鸟

国内外高校在读学生

中小学刚刚入门的新人

在 “未闻 Code 技术交流群” 等你来！

_入群方式：添加微信 “mekingname”，备注 “粉丝群”（谢绝广告党，非诚勿扰！)_

好文和朋友一起看~