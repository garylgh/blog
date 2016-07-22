---
title: Javascrip Date
date: 2016-07-21 19:22:08
tags:
---

1. 一些概念

  UTC：英文「Coordinated Universal Time」／法文「Temps Universel Coordonné」）中文：世界标准时间。简单来说便是0度经线的时间。北京处于东八区，写作UTC+8;
  GMT：格林尼治标准时间，是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间。理论上来说，GMT应该等于UTC。

2. 时间的初始化
  ```
  Date()
  ```     

3. 遇见的坑

  问题描述：项目中使用cookies(https://github.com/pillarjs/cookies)，设置资源的过期时间为8个小时以后，但是发现无论如何设置，浏览器上的cookie expired time都为当前时间。

  原因：阅读代码（v0.6.1）发现，第148行（如下）调用toUTCString()，恰好我们所处的市区为东8区，toUTCString获取时间恰好差8个小时，因此在浏览器永远都是cookies刚返回就失效的状况。

  ```
    if (this.expires  ) header += "; expires=" + this.expires.toUTCString()
  ```

4. 时间的格式化以及不同浏览器的差异
