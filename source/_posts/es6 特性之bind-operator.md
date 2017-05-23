---
title: es6 特性之bind-operator
date: 2017-03-22 15:14:37
tags: es6
---

Usage:
```js
obj::func
// is equivalent to:
func.bind(obj)

obj::func(val)
// is equivalent to:
func.call(obj, val)

::obj.func(val)
// is equivalent to:
func.call(obj, val)
```

+ 参考
  + eslint监测： http://stackoverflow.com/questions/35534580/eslint-and-this-bind-operator
  + 安装依赖：http://babeljs.io/docs/plugins/transform-function-bind/