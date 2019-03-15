---
title: nodejs实现tail -f功能
date: 2019-03-15 09:37:54
categories: nodejs
tags: nodejs, tailf
toc: true
---
```
'use strict';

const fs = require('fs');

/**
 * tailf
 *
 * @param {String} filename 文件名
 * @param {Number} delay 读取不到内容时等待的时间，ms
 * @param {Function} onError 操作出错时的回调函数，onError(err)
 * @param {Function} onData 读取到文件内容时的回调函数，onData(data)
 */
function tailf(filename, delay, onError, onData) {

  // 每次读取文件块大小，16K
  const CHUNK_SIZE = 16 * 1024;
  // 打开文件，获取文件句柄
  fs.open(filename, 'r', (err, fd) => {
    if (err) return onError(err);

    // 文件开始位置
    fs.fstat(fd, (err, stats) => {
      if (err) return onError(err);

      // 文件开始位置
      let position = stats.size;
      // 循环读取
      const loop = () => {

        const buf = Buffer.alloc(CHUNK_SIZE);
        fs.read(fd, buf, 0, CHUNK_SIZE, position,
        (err, bytesRead, buf) => {
          if (err) return onError(err);

          // 实际读取的内容长度以 bytesRead 为准
          // 并且更新 position 位置
          position += bytesRead;
          onData(buf.slice(0, bytesRead));

          if (bytesRead < CHUNK_SIZE) {
            // 如果当前已到达文件末尾，则先等待一段时间再继续
            // setTimeout(loop, delay);
          } else {
            loop();
          }
        });
      };
      loop();
      // 监听文件变化，如果收到 change 事件则尝试读取文件内容
      fs.watch(filename, (event, filename) => {
        if (event === 'change') {
          loop();
        }
      });
    });
  });
}

const filename = process.argv[2];
if (filename) {
  tailf(filename, 100, err => {
      if (err) console.error(err);
    }, data => {
        process.stdout.write(data);
      });
} else {
  console.log('使用方法： node tailf <文件名>');
}
```