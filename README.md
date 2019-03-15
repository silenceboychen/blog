## 基于hexo开发的个人博客项目

使用主题地址:  https://github.com/cofess/hexo-theme-pure

####开发步骤：
hexo项目目录下执行：

```
git clone https://github.com/cofess/hexo-theme-pure themes/pure
```

将主题下载到themes目录下。

hexo项目目录下执行：

```
cp image/wechatpayimg.jpeg ./themes/pure/source/images/donate/
cp image/wechatpayimg.jpeg ./themes/pure/source/images/donate/
cp image/avatar.jpeg ./themes/pure/source/images/
cp themes_config.yml ./themes/pure/_config.yml
```
将自己的配置拷贝进入主题目录下。

创建文章编译并发布

```
$ hexo new [layout] <title>

```
新建一篇文章。如果没有设置 ``layout`` 的话，默认使用 ``_config.yml`` 中的 ``default_layout`` 参数代替。如果标题包含空格的话，请使用引号括起来。

```
$ hexo generate
```
生成静态文件。

```
$ hexo deploy
```
部署网站

```
$ hexo clean
```
清除缓存文件 (db.json) 和已生成的静态文件 (public)。

在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

