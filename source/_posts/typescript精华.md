---
title: typescript精华
date: 2018-06-01 11:36:23
tags:
---
# Typescript精华

### Strict Mode

- noImplicitAny
如下的代码在严格模式下将会报错，需显示指定someArg的类型
```
function log(someArg) { // Error : someArg has an implicit `any` type
  sendDataToServer(someArg);
}
```
- strictNullChecks

下段代码在strict模式下，编译器将提示elem可能为null
```
  const elem = document.getElementById('test');
  elem.innerHTML = 'Hello World';
```