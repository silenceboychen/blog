---
title: 取消conda终端默认激活base虚拟环境
toc: true
date: 2025-04-09 18:20:02
categories: python
tags: python
---

默认情况下，安装完conda后，每次打开终端都会自动激活`base`环境。如果你不习惯这一行为，可以通过修改conda的相关配置，使终端启动时不再默认激活`base`环境。

## ▶️ 临时关闭 auto activate (当前终端有效)

如果你只是想一次性临时地关闭，可以运行：

```bash
conda deactivate
```

但是这种方式仅对当前打开的终端临时生效，打开新的终端窗口时，`base`还是会自动激活。


## ✅ 长期永久关闭默认激活 (推荐)

要永久关闭每次终端自动激活`base`环境，可以运行：

```bash
conda config --set auto_activate_base false
```

以上命令会修改conda配置文件`~/.condarc`，添加如下内容：

```yaml
auto_activate_base: false
```

以后你每次打开新的终端窗口时，`base`将不会再自动激活。


## 📝 如果你后悔了，如何恢复自动激活base？

再次启用自动激活：

```bash
conda config --set auto_activate_base true
```

## 🔍 验证你的配置

执行以下命令验证：

```bash
conda config --show | grep auto_activate_base
```

输出：

```yaml
auto_activate_base: false
```

代表成功地关闭了默认激活行为。


⚠️ **注意**：

- 在关闭默认激活之后，如果想激活base或其他虚拟环境，就需要手动运行：

```bash
conda activate base       #激活base
conda activate env_name   #激活其他环境
```


按照以上方法，你便可以自由控制conda是否默认激活`base`环境了。