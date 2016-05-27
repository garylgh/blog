---
title: Eslint开启ES7语法校验
date: 2016-05-27 11:05:53
tags:
---

## 问题

目前eslint不支持spread object语法，经查发现spread object是ES7规范（还包括class properties,decorators,async/await）

参考：https://github.com/eslint/eslint/issues/2532

## 解决

### 1.方法一：
安装babel-eslint
```
npm install babel-eslint --save-dev
```
配置.eslintrc
```
"parser": "babel-eslint"
```
### 2. 方法二：

配置.eslintrc文件, 开启试验功能
```
{
    "extends": "airbnb",
    # this is for object spread
    "parserOptions": {
        "ecmaFeatures": {
            "experimentalObjectRestSpread": true
        }
    }
}
```

安装babel插件，babel-plugin-transform-object-rest-spread
```
npm install babel-plugin-transform-object-rest-spread --save-dev
```

添加下面一行到.babelrc文件，倘若是fis3里使用babel进行编译的话，则加到对应的babel options里
```
{
    "plugins": ["transform-object-rest-spread"]
}
```

To be continue...
