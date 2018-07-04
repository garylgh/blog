---
title: mac安装protobuf2.5.0
date: 2018-06-06 16:48:03
tags:
---
### 起因
编译hadoop2.9.0时提示如下错误：
```
[ERROR] Failed to execute goal org.apache.hadoop:hadoop-maven-plugins:2.9.0:protoc (compile-protoc) on project hadoop-common: org.apache.maven.plugin.MojoExecutionException: protoc version is 'libprotoc 3.5.1', expected version is '2.5.0' -> [Help 1]
```

### 安装步骤
- 下载源码
```bash
wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.bz2
```

- 解压源码
```bash
tar xfvj protobuf-2.5.0.tar.bz2
```

- 配置编译选项
```bash
cd protobuf-2.5.0
./configure CC=clang CXX=clang++ CXXFLAGS='-std=c++11 -stdlib=libc++ -O3 -g' LDFLAGS='-stdlib=libc++' LIBS='-lc++ -lc++abi'
```

- 编译安装
```bash
make -j 4
sudo make install
```