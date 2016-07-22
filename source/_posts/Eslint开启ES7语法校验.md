---
title: Eslint开启ES7语法校验
date: 2016-05-27 11:05:53
tags:
---

## 问题

目前eslint只支持ES6/ES2015，不支持spread object语法，经查发现spread object是ES7规范（还包括class properties,decorators,async/await）

参考：https://github.com/eslint/eslint/issues/2532

## 解决

### 1. eslint语法检查：
安装babel-eslint
```
npm install babel-eslint --save-dev
```
倘若需要支持generator-star，object-shorthand等特性，需要安装eslint-plugin-babel
```
npm install eslint-plugin-babel --save-dev
```
然后在配置文件.eslintrc.* 添加如下配置
```
{
  "parser": "babel-eslint",
  "plugins": [
    "babel"
  ]
  # this is for object spread
  "parserOptions": {
      "ecmaFeatures": {
          "experimentalObjectRestSpread": true
      }
  }
}
```

#### 注意：
此处有一个坑，倘若extend airbnb-base的规则，需要把generator-star-spacing规则off掉。
具体在eslint-plugin-babel的gihub页面上有提及——“Finally enable all the rules you would like to use (remember to disable the original ones as well!).”
```
{
  'rule': {
    'generator-star-spacing': 0, // 需要把airbnb-base定义的这条规则off掉，否则babel-eslint在parse时会报错
  }
}
```

### 2. 代码运行：
上一步只是eslint代码的语法检查配置，倘若需要代码能够执行ES7特性的代码，还需要添加一下babel插件。

支持object-rest-spread特性，安装babel插件，babel-plugin-transform-object-rest-spread
```
npm install babel-plugin-transform-object-rest-spread --save-dev
```

支持特性async/await, 安装如下插件：
```
npm install babel-plugin-transform-runtime --save-dev
npm install babel-plugin-syntax-async-functions --save-dev
npm install babel-plugin-transform-async-to-module-method --save-dev
```

配置.babelrc文件，倘若是fis3里使用babel进行编译的话，则加到对应的babel options里
```
{
    "presets": ["es2015", "stage-3"],
    "plugins": [
      "transform-object-rest-spread",
      "transform-runtime",
      "syntax-async-functions",
      ["transform-async-to-module-method", {
        "module": "bluebird",
        "method": "coroutine"
      }]
    ]
}

```
具体可参考stackoverflow的帖子：http://stackoverflow.com/questions/28708975/transpile-async-await-proposal-with-babel-js
