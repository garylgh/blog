---
title: Nodejs调用c++扩展
date: 2018-06-08 15:19:54
tags:
---
## 相关概念
* V8
  the C++ library Node.js currently uses to provide the JavaScript implementation. [V8 doc](https://v8docs.nodesource.com/)

* libuv
  The C library that implements the Node.js event loop, its worker threads and all of the asynchronous behaviors of the platform. [libuv doc](https://github.com/libuv/libuv)

* Internal Node.js libraries
  Node.js itself exports a number of C++ APIs that Addons can use — the most important of which is the node::ObjectWrap class. 位于node源码的src目录下。

* node-gyp
  a tool written specifically to compile Node.js Addons. This version is not made directly available for developers to use and is intended only to support the ability to use the npm install command to compile and install Addons.
  [Linking to Node.js' own dependencies](https://nodejs.org/api/addons.html#addons_linking_to_node_js_own_dependencies)
  [node-gyp doc](https://github.com/nodejs/node-gyp#installation)

  when node-gyp runs, it will detect the node's version, and find the sources or headers in ~/.node-gyp. If it cannot find any sources or headers, it will download  either the full source tarball or just the headers. We can use the --nodedir flag pointing at a local Node.js source image.

* binding.gyp
  node-gyp在编译时，会读取addon根目录下的binding.gyp文件，然后build/目录下生产Makefile文件(Unix/MacOs)或者vcxproj文件(Windows)，文件结构如下：
```
  {
    "targets": [
      {
        "target_name": "hello",
        "sources": [ "hello.cc" ],
        "include_dirs": [
          "<!(node -e \"require('nan')\")"
        ]
      },{
        "target_name": "addon2",
        "sources": [ "2/addon.cc", "2/myobject.cc" ]
      }
    ]
  }
```

* .node
  .node文件是c++ addon经node-gyp编译后生成的动态链接库(相当于.dll或.so或.dylib)，一般位于build/Release/addon.node, 可供js调用. 底层通过process.dlopen(module, filename)函数加载(in node.cc)。
  ```
  const addon = require('./build/Release/addon')
  addon.your_c_function()
  ```
  如果引入bindings包（见下节）可调用如下：
  ```
  const addon = require('bindings')('addon.node')
  addon.your_c_function()
  ```

* node-bindings
  [node-bindings doc](https://github.com/TooTallNate/node-bindings)

* node-ffi
  node-ffi is a Node.js addon for loading and calling dynamic libraries using pure JavaScript. It can be used to create bindings to native libraries without writing any C++ code.

* Nan(Native Abstractions for Node.js)
  A header file filled with macro and utility goodness for making add-on development for Node.js easier across versions 0.8, 0.10, 0.12, 1, 2, 3, 4, 5, 6, 7, 8, 9 and 10.
  [doc](https://github.com/nodejs/nan)

* N-API(From v8.0.0)
  N-API (pronounced N as in the letter, followed by API) is an API for building native Addons. It is independent from the underlying JavaScript runtime (ex V8) and is maintained as part of Node.js itself. This API will be Application Binary Interface (ABI) stable across versions of Node.js. It is intended to insulate Addons from changes in the underlying JavaScript engine and allow modules compiled for one version to run on later versions of Node.js without recompilation.
  [doc](https://nodejs.org/api/n-api.html)

* node-addon-api
  [doc](https://github.com/nodejs/node-addon-api)

## 实现的三种方式
  1. 直接使用Node.js和V8 API
    -缺点：V8 API can, and has, changed dramatically from one V8 release to the next

  2. Nan

  3. 使用N-API

  4. 使用node-ffi

## 案例分析
  [sharp](https://github.com/lovell/sharp)是一个Node.js的图像处理库，底层依赖c++图像库[libvips](https://github.com/jcupitt/libvips),下面通过分析sharp的构建过程，学习如何编写一个c++ addon

  * 相关资源
    https://github.com/lovell/sharp
    https://github.com/lovell/sharp-libvips
    https://github.com/lovell/package-libvips-darwin
    https://github.com/jcupitt/libvips
  * 安装过程
    1. 执行npm install，将执行以下指令
    ```bash
    (node install/libvips && node install/dll-copy && prebuild-install) || (node-gyp rebuild && node install/dll-copy)
    ```
    其中install/libvips.js将根据platform和版本，从https://github.com/lovell/sharp-libvips/releases下载libvips依赖的动态链接库，此库将保存在vendor目录下，包含所有的动态链接库和头文件。sharp-libvips工程通过docker将不同平台依赖的包预先构建打包了。
    2. 执行node-gyp rebuild
    此命令将根据binding.gyp文件的描述，将c代码编译，生成目标文件build/Release/sharp.node。至此安装完成
  

## 参考
  [C++ Addons](https://nodejs.org/api/addons.html)
  [Node.js 原生模块开发方式变迁](https://cnodejs.org/topic/5957626dacfce9295ba072e0)
  [N-API: Next generation Node.js APIs for native modules](https://medium.com/the-node-js-collection/n-api-next-generation-node-js-apis-for-native-modules-169af5235b06)