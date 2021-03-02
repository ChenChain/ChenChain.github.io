---
title: anti anti spider concept
top: false
toc: true
date: 2021-03-02 16:36:53
password:
summary:
categories: 边城
tags: spider
---
### Anti-Anti-Spider Concept



> 最近因为毕业设计 需要对网络资源进行爬取。记录爬虫的规则与反爬 (网络不是非法之地~
>
> 不做任何盈利目的与网站攻击目的



#### robots.txt

每个网站是否可以被爬，以及被爬的范围，都定义在robots.txt。如果一个网站上没有robots.txt，是被认为默许爬虫爬取所有信息。一个自觉且善意的爬虫，应该在抓取网页之前，先阅读robots.txt，了解并执行网站管理者制定的爬虫规则。

在浏览器输入`根域名/robots.txt`即可查看爬虫范围。例如`https://www.zhihu.com/robots.txt`页面：

```
User-agent: Googlebot // 爬虫的名称，若是*，则允许所有爬虫
Disallow: /search // 不允许爬虫访问的地址
Allow: /search-special //允许爬虫访问的地址
Disallow: /*?guide*

User-agent: Googlebot-Image
Disallow: /search
Allow: /search-special
Disallow: /*?guide* // regex规则匹配

...
```



#### 代理

- 原理

本机与服务器之间增加代理服务器，本机通过代理服务器来访问服务器，形成IP伪装

- 作用
    - 突破IP限制，访问该IP不能访问的站点
    - 隐藏真实IP

- 爬虫代理

爬虫在爬取过程中不断更换代理，来更换不同的IP。例如分布式爬虫主要的一个作用就是不同IP代理爬取资源。

- 代理分类
    - FTP代理服务器，用来访问FTP服务器
    - HTTP代理服务器，用来访问网页
    - SSL/TLS代理服务器，用来访问加密网页

#### 反爬技术

日常上网中遇到的滑动拼图，短信验证码等都属于反爬技术，用来人机验证，阻止爬虫批量爬取网站内容。当然，反爬可能会误伤：导致普通用户无法访问。当网站发现一个IP访问过于频繁，它就会开始反爬策略，要求我们输入验证码或者登陆或直接封ip。



常见反爬策略：

- user-agent识别

  有些爬虫的UA是特殊的，与正常浏览器不一样，可以通过识别UA特征，拒绝该请求。简单来说是终端的环境信息如：Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/533.21.1 (KHTML, like Gecko) Version/5.0.5 Safari/533.21.1。但是这只是一般意义上的ua参数，实际使用中ua代表了一个终端的标识，ID代表了一个用户的标识，随着网络安全的深入，ua已经不再是一串环境的信息字符串，而是开发者单独发开出来的一套甄别终端的算法，成为了单独执行的js文件。一般在requests的headers里面我们都会附带上User_Agent以规避后台检测是否为爬虫的请求，但是随着安全的提升现在的大型网站已经不再用请求头里面的ua，而是使用单独的js本地生成ua参数，再发给浏览器然后对当前的终端进行鉴别。

- IP访问频率

    1. 当IP访问次数超过阀值，则要求登陆或验证码输入。验证码输入错误则禁止请求一段时间，超过一定次数错误则直接封IP。

    2. 爬虫访问的IP并发都很高，统计IP并发次数来封禁IP
    3. IP访问的时间窗口统计来封禁：爬虫爬取网页的频率都是比较固定的，不像人去访问网页，中间的间隔时间比较无规则，所以我们可以给每个IP地址建立一个时间窗口，记录IP地址最近12次访问时间，每记录一次就滑动一次窗口，比较最近访问时间和当前时间，如果间隔时间很长判断不是爬虫，清除时间窗口，如果间隔不长，就回溯计算指定时间段的访问频率，如果访问频率超过阀值，就转向验证码页面让用户填写验证码
    4. 限制单个IP的访问量：比如10分钟至多允许访问该页面100次

- 识别合法爬虫

  一些网站希望被百度，谷歌爬虫爬取。他们会通过识别UA与IP的对应关系(百度/谷歌等爬虫的IP都可以在网络上找到)。

- 蜜罐资源

  爬虫会爬取一些浏览器上访问不到的资源，对访问蜜罐资源的ip统计，进行封禁

- 以特定的参数和密钥生成token，请求需要带上这个token交给服务器验证

- html，css字体替换。如：机票`<span class="kw_01">`元。那么kw_1就需要html渲染后替换成真实的数字。



反反爬策略：

- 设置爬取/下载资源延迟，延迟越大越安全
- 禁用cookie：一些网站使用cookie识别用户身份，禁用后服务器无法跟踪爬虫轨迹
- 变换UA，IP
- 浏览器行为模拟：如selenium工具等
- OCR识别，训练深度学习模型识别验证码（比如使用 CNN）或者直接对接打码平台绕过验证码输入；手机验证码接码平台绕过
- 分析网页html,js代码，分析出使用的token加密过程，自己构造token去请求api


爬虫，最困难的部分，也就是反反爬了吧～


#### 一些算法

- **BloomFilter**

  用于海量URL过滤去重



#### 一些工具

- python框架

    - scrapy：强大爬虫框架，支持分布式与scrapy-redis

    - selenium：可以模拟人类浏览浏览器的动作。使用该工具时会有特殊标示，如webdriver=true，这一点会被反爬

    - faker: 随机user-agent生成库
    - plantomJS：无界面浏览器

- Charles：请求抓包；API分析
- Chrome：debug调试；API分析



#### 参考

[Google Robots.txt规范](https://developers.google.com/search/reference/robots_txt?hl=zh-cn)

[selenium doc](https://www.selenium.dev/documentation/zh-cn/getting_started_with_webdriver/)

[知乎-反爬虫策略故事](https://www.zhihu.com/question/28168585)

[知乎-反反爬实践example](https://zhuanlan.zhihu.com/p/55187714)

[知乎-淘宝反反爬](https://zhuanlan.zhihu.com/p/55641629)

