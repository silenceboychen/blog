---
title: fidl和arxml互转教程
toc: true
date: 2025-09-05 17:56:31
categories: autosar
tags: [fidl, arxml]
---

``fracon`` 命令行工具可以使用 ``maven`` 从源代码构建。本页介绍如何运行本地构建。 请注意，你的公司需要成为 ``AUTOSAR`` 组织的成员。如果不是会员，你将无法访问 ``artop.org``，因此无法使用 FARACON 工具。

环境要求：java8 / mvn3.x

注意：**以下所有操作均在ubuntu20.04上进行。**

## 如何从源码构建命令行工具

### 源码下载

源码地址：<https://github.com/COVESA/franca_ara_tools>

```
$ git clone https://github.com/GENIVI/franca_ara_tools.git
```

### artop文件下载

为了构建`franca_ara_tools`项目，需要下载`artop`提供的`artop-Update-4.12.1`文件。只有加入`autosar`会员的公司才能注册和登录`artop`站点：<https://www.artop.org>。

1.  登录`artop`站点后，选择`Downloads`，然后选择`All Downloads`。

![](/../images/artop1.png)

2.  选择`SDK`

![](/../images/artop2.png)

3.  下拉找到`artop-Update-4.12.1.zip`

**注意：目前只能使用4.12.1版本，其他版本无效。**

![](/../images/artop3.png)

4.  `artop-Update-4.12.1.zip`解压到制定目录：

```
$ unzip -d /home/test/franca/artop-Update-4.12.1 artop-Update-4.12.1.zip
```

### franca_ara_tools源码修改

进入下载的franca_ara_tools项目目录，然后执行以下操作：

1.  修改`releng/org.genivi.faracon.parent/pom.xml`文件

```
+++ b/releng/org.genivi.faracon.parent/pom.xml
@@ -401,6 +401,16 @@
                                                        <groupId>org.eclipse.jdt</groupId>
                                                        <artifactId>org.eclipse.jdt.core</artifactId>
                                                        <version>${jdt-core-version}</version>
+                                                       <exclusions>
+                                                               <exclusion>
+                                                                       <groupId>org.eclipse.platform</groupId>
+                                                                       <artifactId>org.eclipse.core.runtime</artifactId>
+                                                               </exclusion>
+                                                               <exclusion>
+                                                                       <groupId>org.eclipse.platform</groupId>
+                                                                       <artifactId>org.eclipse.equinox.common</artifactId>
+                                                               </exclusion>
+                                                       </exclusions>
                                                </dependency>
                                                <dependency>
                                                        <groupId>org.eclipse.jdt</groupId>
@@ -412,10 +422,36 @@
                                                        <artifactId>org.eclipse.jdt.compiler.tool</artifactId>
                                                        <version>${jdt-compiler-tool-version}</version>
                                                </dependency>
+                                               <dependency>
+                                                       <groupId>org.eclipse.platform</groupId>
+                                                       <artifactId>org.eclipse.core.runtime</artifactId>
+                                                       <version>3.12.0</version>
+                                                       <exclusions>
+                                                               <exclusion>
+                                                                       <groupId>org.eclipse.platform</groupId>
+                                                                       <artifactId>org.eclipse.equinox.common</artifactId>
+                                                               </exclusion>
+                                                       </exclusions>
+                                               </dependency>
                                                <dependency>
                                                        <groupId>org.eclipse.emf</groupId>
                                                        <artifactId>org.eclipse.emf.codegen</artifactId>
                                                        <version>${emf-codegen-version}</version>
+                                                       <exclusions>
+                                                               <exclusion>
+                                                                       <groupId>org.eclipse.platform</groupId>
+                                                                       <artifactId>org.eclipse.core.runtime</artifactId>
+                                                               </exclusion>
+                                                               <exclusion>
+                                                                       <groupId>org.eclipse.platform</groupId>
+                                                                       <artifactId>org.eclipse.equinox.common</artifactId>
+                                                               </exclusion>
+                                                       </exclusions>
+                                               </dependency>
+                                               <dependency>
+                                                       <groupId>org.eclipse.platform</groupId>
+                                                       <artifactId>org.eclipse.equinox.common</artifactId>
+                                                       <version>3.8.0</version>
                                                </dependency>
```

![](/../images/fidl1.png)

2.  修改`releng/org.genivi.faracon.target/fara-oxygen-artop.target`文件，将location修改为本地artop文件目录：`<repository location="file:/home/test/franca/artop-Update-4.12.1"/>`

![](/../images/fidl2.png)

### franca_ara_tools源码构建

进入`franca_ara_tools`项目的`releng/org.genivi.faracon.parent`目录，执行以下命令：

```
$ mvn clean install -Pwith-artop -Dtycho.disableP2Mirrors=true
```

等待构建完成，时间较长：

![](/../images/fidl3.png)

构建成功之后在`franca_ara_tools`项目下生成products目录，构建成果物在该目录下。

## 文件转换

构建完成后在`products/org.genivi.faracon.cli.product/target/products/org.genivi.faracon.cli.product`目录下会生成各个平台的可执行文件，进入目录：

![](/../images/fidl4.png)

```
$ cd products/org.genivi.faracon.cli.product/target/products/org.genivi.faracon.cli.product
```

由于我在`ubuntu20.04`下操作，将对应系统文件`copy`到指定目录：

```
$ cp -a franca_ara_tools/products/org.genivi.faracon.cli.product/target/products/org.genivi.faracon.cli.product/linux/gtk/x86_64 $HOME/faracon
```

为了在任意地方使用`faranca`命令，设置环境变量：`vim ~/.zshrc`,添加以下内容：

```
export PATH=$PATH:$HOME/faracon
```

### 命令行选项

`faracon-linux-x86_64 --help` 将输出以下内容：

```
Command: Console Help
Command: Franca ARA Converter
usage: faracon [-a <arg>] [-c] [-ca] [-d <arg>] [-e] [-f <arg>] [-i <arg>] [-L
       <arg>] [-l <arg>]
 -a,--ara-to-franca <arg>       Arxml file that will be converted to .fidl or
                                directory what will be recursively scanned for
                                .arxml files and each one of them will be
                                converted to corresponding fidl ones.
 -c,--continue-on-errors        Do not stop the tool execution when an error
                                occurs.
 -ca,--check-arxml-files-only   Checks the provided ARXML files.
 -conf,--f2aconfig <arg>        Supply configuration file for command line
                                related to Franca-to-arxml generation.
 -d,--dest <arg>                Output directory for the generated files.
 -e,--warnings-as-errors        Treat warnings as errors.
 -f,--franca-to-ara <arg>       Franca file that will be converted to arxml or
                                directory what will be recursively scanned for
                                fidl files and each one of them will be
                                converted to corresponding arxml ones.
 -i,--include <arg>             Additions to classpath.
 -L,--license <arg>             The file path to the license text that will be
                                added to each generated file.
 -l,--log-level <arg>           The log level (quiet or verbose).

Command: Console Help
usage: faracon -h
 -h,--help   Print out options of the tool.

Command: Version Information
usage: faracon -v
 -v,--version   Show version number of the tool.
```

### 转换前注意事项

终于可以使用命令行工具了，激动的去试一试，但是发现在执行转换命令操作的时候遇到以下报错：

![](/../images/fidl5.png)

经过一番排查之后，找到了问题原因，修改`$HOME/faracon/faracon-linux-x86_64.ini`文件内容，删除`--illegal-access=deny`参数。

删除前：

![](/../images/fidl6.png)

删除后：

![](/../images/fidl7.png)

之后就可以按照以下步骤进行转换了。👦

### fidl转arxml

转换 `/path/to/fidls` 中的所有 `fidl` 文件，将输出存储在`  /path/to/output  `中，使用详细的日志记录级别，并在发生错误时继续翻译下一个 `fidl`：

转换`fidl`使用参数`-f`

```
$ faracon-linux-x86_64 -f /path/to/fidls/ -d /path/to/output/ -l verbose -c
```

也可指定文件：

```
$ faracon-linux-x86_64 -f /path/to/fidls/helloworld.fidl -d /path/to/output/ -l verbose -c
```

![](/../images/fidl8.png)

### arxml转fidl

转换`  /path/to/arxmls ` 中的所有 `arxml` 文件，将输出存储在`  /path/to/output ` 中，使用详细的日志记录级别，并在发生错误时继续翻译下一个 `arxml`：

转换`arxml`使用参数`-a`

```
$ faracon-linux-x86_64 -a /path/to/arxmls/ -d /path/to/output/ -l verbose -c
```

也可指定文件：

```
$ faracon-linux-x86_64 -a /path/to/arxmls/helloworld.arxml -d /path/to/output/ -l verbose -c
```

![](/../images/fidl9.png)