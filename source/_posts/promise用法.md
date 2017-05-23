---
title: promise用法
date: 2016-08-05 11:04:10
tags: es6
---
## 链式调用和书写风格

`then` 可以使用链式调用的写法原因在于，每一次执行该方法时总是会返回一个 Promise 对象。

<!-- more -->

```javascript
var flag1 = false;
var getPromise1 = () => {
	return new Promise((resolve, reject) => {
		flag1 ? resolve("promise1 resolved") : reject("promise1 rejected");//....异步操作

	});
};
var getPromise2 = Promise.resolve('foo');// 推荐的写法 
```

如何在`onRejected`中抛出错误?

 - `throw("something err")` 
 - `return Promise.reject("ERROR!!!")`

```javascript
getPromise1()
	.then(
		resolve => resolve
		,reject => reject
	).then(
		resolve => {
			return ("1.resolved +"+resolve)
		},reject => {
			// return "error" 不能冒泡到下个reject 或 被catch捕获
			// throw ("ERROR!") OK! 
			return Promise.reject("ERROR!!!")
		}
	).catch(
		err => console.log("err:"+err)// 推荐的写法

	)
```

## Promise.all 和 Promise.race

`Promise.all` 可以接收一个元素为 `Promise` 对象的数组作为参数，当这个数组里面所有的 Promise 对象都变为 `resolve` 时，该方法才会返回。

```javascript
var p1 = new Promise((resolve) => {
    setTimeout(() => {
        resolve("Hello"); 
    }, 3000);
});

var p2 = new Promise((resolve) => {
    setTimeout(() => {
        resolve("World");
    }, 1000);
});

Promise.all([p1, p2]).then((result) => {
    console.log(result); // ["Hello", "World"]
});

```

`Promise.race` 可以接收一个元素为 `Promise` 对象的数组作为参数，不同的是数组里面所有的 Promise 对象都**改变状态**时，方法即返回。