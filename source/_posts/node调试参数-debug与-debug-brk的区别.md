---
title: node调试参数--debug与--debug-brk的区别
date: 2016-07-25 18:15:50
tags:
---

#### 问题

  在启动项目调试时，发现web服务器启动极慢，慢到无法忍受。经查找发现在node启动时--debug-brk参数惹的祸。当使用--debug-brk启动调试时，程序将停留在第一行，而且需要等待调试工具连接以后（即http://127.0.0.1:8080/?port=5858返回好久以后，此处还有疑问，为啥devTools已经加载完代码很久了，服务端代码才继续执行下去），服务端代码才正常启动起来，这样浏览器才能正常访问页面。

  因此启动时改用--debug参数，这样在node --debug server命令运行完成后，服务器端的代码都会自动运行完成。浏览器就能正常访问页面了。3Q


#### 区别

  **--debug-brk** stops the node program on the first line, meaning it will break before starting the server. You can then connect your debugger and hit Continue to run the program.

  **--debug**  start the debugger but not break at the start. So if you have a debugger line somewhere in asynchronous code, it will still break when it hits it if your debugger is connected.




#### 参考
  http://stackoverflow.com/questions/20384822/node-debug-brk-app-js-not-functioning
