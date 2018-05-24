---
title: 使用SSH Tunnel远程调试Nodejs
date: 2018-05-24 17:01:24
tags:
---


### 环境
  1. 远程机器
```
  host: xx.xx.96.106
  port: 9000
  wsport: 9229
```
  2. 本地机器
```
  host: localhost
  port: 9221
```

### 步骤
  1. 启动远程服务
```
  node --inspect server/app.js
	# 输出如下信息说明node inspector成功启动
  # Debugger listening on ws://127.0.0.1:9229/f396b0da-7312-48cd-a933-1ea6f7c1c197
```

  2. 启动本地SSH Tunnel
```
  ssh -L 9221:localhost:9229 xiaoju@10.94.96.106
```

  3. 启动debugger client，入chrome dev tools 或者 vscode，此处使用dev tools

    浏览器地址栏输入chrome://inspect，点击“configure”按钮，输入“loclahost:9221”，点击“done”按钮，成功启动dev tools

  4. 在dev tools里给想访问的代码打上断点，通过浏览器访问远程接口，代码将终止在断点处
  
    此处访问：http://xxx.xxx.com:9000/#/list

### 参考
  https://nodejs.org/en/docs/guides/debugging-getting-started/
  http://blog.creke.net/722.html
