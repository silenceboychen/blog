---
title: '使用npm安装依赖时报错：gyp: No Xcode or CLT version detected!'
toc: true
date: 2020-05-05 16:47:52
categories: npm
tags: npm
---

最近在macOS中使用npm安装模块，出现如下错误：

```
npm WARN deprecated fsevents@1.2.12: fsevents 1 will break on node v14+. Upgrade to fsevents 2 with massive improvements.

> fsevents@1.2.12 install /Users/chenhao/outsourcing/egg-car/node_modules/fsevents
> node-gyp rebuild

No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.

gyp: No Xcode or CLT version detected!
gyp ERR! configure error
gyp ERR! stack Error: `gyp` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onCpExit (/Users/chenhao/.nvm/versions/node/v10.16.0/lib/node_modules/npm/node_modules/node-gyp/lib/configure.js:351:16)
gyp ERR! stack     at ChildProcess.emit (events.js:198:13)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:248:12)
gyp ERR! System Darwin 19.4.0
gyp ERR! command "/Users/chenhao/.nvm/versions/node/v10.16.0/bin/node" "/Users/chenhao/.nvm/versions/node/v10.16.0/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /Users/chenhao/outsourcing/egg-car/node_modules/fsevents
gyp ERR! node -v v10.16.0
gyp ERR! node-gyp -v v5.0.5
gyp ERR! not ok
```

网上查找解决方案，都是通过执行`xcode-select --install`命令修复，但是执行该命令时会出现如下提示：

```
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

最终的解决办法是先卸载之前安装的`xcode-select`，并重新安装：

```
$ sudo rm -rf $(xcode-select -print-path)
$ xcode-select --install
```