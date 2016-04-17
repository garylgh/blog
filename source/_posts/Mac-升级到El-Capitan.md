---
title: Mac 升级到El Capitan
date: 2016-04-15 10:10:06
tags: Mac
---

### 问题

    通过App Store跟新El Capitan，过程奇慢无比，几乎是不可能完成。下面是可行的更新办法。

### 更新步骤

1. 抓取升级包的下载链接
    目前网上有提供升级包的下载链接，但是几乎都出自同一篇文章，且该链接已经过时。为保险起见，还是自己手动抓包比较靠谱。抓包工具可选wireshark或Charles。因为本机只装了wireshark，因此用wireshark抓取，选中网卡eth0，Capture Filter选中TCP or UDP port 80 (HTTP)。打开App Store，点击升级El Capitan。抓取到链接如下：

    ```
    http://osxapps.itunes.apple.com/apple-assets-us-std-000001/Purple49/v4/de/57/69/de576928-f6f5-9384-8dd4-974e3fcca649/encrypted226869791772508350.pkg
    ```

2. 使用迅雷等工具下载刚才抓取的pkg包

3. 搭建本地代理服务器
    这步的作用是将App Store更新时将目标服务器指向本地，具体步骤如下：
    ```
    #create a tmp folder
    mkdir elCapitanRoot && cd elCapitanRoot

    #create a folder structure to match apple server
    sudo mkdir -p ./apple-assets-us-std-000001/Purple49/v4/de/57/69/de576928-f6f5-9384-8dd4-974e3fcca649/

    #move downloaded pkg file to proper path
    sudo mv ../encrypted226869791772508350.pkg ./apple-assets-us-std-000001/Purple49/v4/de/57/69/de576928-f6f5-9384-8dd4-974e3fcca649/

    #start a web server when you are in "elCapitanRoot" folder
    sudo python -m SimpleHTTPServer 80

    #修改hosts文件
    sudo echo "127.0.0.1 	osxapps.itunes.apple.com" >> /etc/hosts
    ```
    以上步骤完成之后，就可以到App Store里重新升级了，此时就能看见更新速度和火箭一般。更新完成之后，记得把hosts改回来，不然以后就没法更新其它软件了。

4. 至此就进入常规步骤，整个安装过程大概会花费30分钟吧。

5. 题外话：在第一步时还经常会遇见更新软件时经常碰到的坑：“进入已购项目，但是提示请等待，无法下载。也无法取消 ”，这就需要删除本地缓存，然后重新更新。清除缓存步骤如下：

    1. 强制退出App Store
    2. 进入下载缓存目录，在终端输入：
    ```
    sudo open $TMPDIR/../C/
    ```
    3. 直接删除com.apple.appstore 目录
    4. 启动 Mac的 App Store，重新下载。

### 参考

    https://gist.github.com/rahul286/2fc41942c7ed4039893f
    http://www.jianshu.com/p/3cb89725da64
