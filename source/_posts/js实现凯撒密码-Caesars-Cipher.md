---
title: js实现凯撒密码 (Caesars Cipher)
toc: true
date: 2019-04-04 17:31:04
categories: 算法
tags: [算法, javascript]
---

> 凯撒密码Caesar cipher，又叫移位密码。移位密码也就是密码中的字母会按照指定的数量来做移位。
> 一个常见的案例就是ROT13密码，字母会移位13个位置。由'A' ↔ 'N', 'B' ↔ 'O'，以此类推。

所有的字母都是大写，不要转化任何非字母形式的字符(例如：空格，标点符号)，遇到这些特殊字符，跳过它们。

判断是否为大写也不难，我们可以通过 .charCodeAt() 返回的 ASCII 码来判断

至于加密的实现，我们可以像这样分情况讨论：

* 如果当前字符为 A - M 之间，对应的 ASCII 码范围就是 65 - 77，那么 ROT13 加密应该给它的 ASCII 码加 13
* 如果当前字符为 N - Z 之间，对应的 ASCII 码范围就是 78 - 90，那么 ROT13 加密应该给它的 ASCII 码减 13
* 如果当前字符为其他 (小写，空格或特殊符号)，那就不应该执行任何操作

```
function rot13(str) {
    let result = "";

    for (let i = 0; i < str.length; i++) {
        const currentCode = str[i].charCodeAt();
        if (currentCode > 90 || currentCode < 65) {
            // 非大写字符
            result += String.fromCharCode(currentCode);
        } else if (currentCode < 78) {
            // 大写字符 A - M
            result += String.fromCharCode(currentCode + 13);
        } else {
            // 大写字符 N - Z
            result += String.fromCharCode(currentCode - 13);
        }
    }

    return result;
}
```

通过求余数来进行优化：

```
function rot13(str) {
    return str.replace(/[A-Z]/g, char => String.fromCharCode(char.charCodeAt() % 26 + 65));
}
```