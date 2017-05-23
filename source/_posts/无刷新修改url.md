---
title: 无刷新修改url
date: 2016-08-26 15:14:37
tags: 代码段
---

先修改后替换

<!-- more -->

```javascript
/**
 * 修改url
 * @param  {String} destiny   目标字符串
 * @param  {String} par       参数名
 * @param  {String} par_value 参数更改的值
 * @return {String}           修改后的url 字符串
 */
changeURLPar(destiny, par, par_value) {
    var pattern = par + '=([^&]*)';
    var replaceText = par + '=' + par_value;
    if (destiny.match(pattern)) {
        var tmp = '/\\' + par + '=[^&]*/';
        tmp = destiny.replace(eval(tmp), replaceText);
        return (tmp);
    } else {
        if (destiny.match('[\?]')) {
            return destiny + '&' + replaceText;
        } else {
            return destiny + '?' + replaceText;
        }
    }
    return destiny + '\n' + par + '\n' + par_value;
},
/**
 * 替换url
 * @param  {String} newUrl 新的url
 * @return {void}       
 */
changeURL(newUrl){
    window.history.pushState({},0,newUrl);      
}
```