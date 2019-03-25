---
title: JSON.stringify 函数参数分析
toc: true
date: 2016-07-16 09:50:57
categories: javascript
tags: javascript
---

``JSON.stringify``是将 JavaScript 值转换为 JavaScript 对象表示法 。
语法为：``JSON.stringify(value [, replacer] [, space])``
很多人都只会用到第一个参数，所以导致很多人不知道后两个参数是什么意思，下面对三个参数进行分析：

**value**
必需。 要转换的 JavaScript 值（通常为对象或数组）。

**replacer**
可选。 用于转换结果的函数或数组。 
如果 ``replacer`` 为函数，则 ``JSON.stringify`` 将调用该函数，并传入每个成员的键和值。 使用返回值而不是原始值。 如果此函数返回 ``undefined``，则排除成员。 根对象的键是一个空字符串：""。 
如果 ``replacer`` 是一个数组，则仅转换该数组中具有键值的成员。 成员的转换顺序与键在数组中的顺序一样。 当 ``value`` 参数也为数组时，将忽略 ``replacer`` 数组。

**space**
可选。 向返回值 JSON 文本添加缩进、空格和换行符以使其更易于读取。 
如果省略 ``space``，则将生成返回值文本，而没有任何额外空格。
如果 ``space`` 是一个数字，则返回值文本在每个级别缩进指定数目的空格。 如果 ``space`` 大于 10，则文本缩进 10 个空格。 
如果 ``space`` 是一个非空字符串（例如“t”），则返回值文本在每个级别中缩进字符串中的字符。
如果 ``space`` 是长度大于 10 个字符的字符串，则使用前 10 个字符。