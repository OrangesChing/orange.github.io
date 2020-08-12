---
title: Hexo常用命令
date: 2019-10-04 10:59:00
tags: 
	- 博客
---

详细使用文档参见 [hexo](https://hexo.io/zh-cn/) 官网

[TOC]

# 新建网站 init

```
$ hexo init [路径]
```

新建一个网站。如果没有设置 `folder` ，Hexo 默认在当前文件夹建立网站。

# 新建文章 new

```
$ hexo new [布局][参数] "文章名字" 
```

新建一篇文章。如果没有设置 `layout` 的话，默认使用 [_config.yml](https://hexo.io/zh-cn/docs/configuration) 中的 `default_layout` 参数代替

# 生成静态网站 generate

```
$ hexo generate
```

生成静态文件。

| 选项             | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| `-d`, `--deploy` | 文件生成后立即部署网站                                       |
| `-w`, `--watch`  | 监视文件变动                                                 |
| `-b`, `--bail`   | 生成过程中如果发生任何未处理的异常则抛出异常                 |
| `-f`, `--force`  | 强制重新生成文件 Hexo 引入了差分机制，如果 `public` 目录存在，那么 `hexo g` 只会重新生成改动的文件。 使用该参数的效果接近 `hexo clean && hexo generate` |

该命令可以简写为

```
$ hexo g
```

# 发布草稿publish

```
$ hexo publish [layout] <filename>
```

发表草稿。

# 启动服务器 server

```
$ hexo server
```

启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。

| 选项             | 描述                           |
| :--------------- | :----------------------------- |
| `-p`, `--port`   | 重设端口                       |
| `-s`, `--static` | 只使用静态文件                 |
| `-l`, `--log`    | 启动日记记录，使用覆盖记录格式 |

# 部署网站deploy

```
$ hexo deploy
```

部署网站。

| 参数               | 描述                     |
| :----------------- | :----------------------- |
| `-g`, `--generate` | 部署之前预先生成静态文件 |

该命令可以简写为：

```
$ hexo d
```

# 清除缓存 clean

```
$ hexo clean
```

清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

在某些情况（尤其是更换主题后），如果发现对站点的更改无论如何也不生效，可能需要运行该命令。

# 查看版本 version

```
$ hexo version
```

显示 Hexo 版本。

# 各种模式

## 安全模式

```
$ hexo --safe
```

在安全模式下，不会载入插件和脚本。当您在安装新插件遭遇问题时，可以尝试以安全模式重新执行。

## 调试模式

```
$ hexo --debug
$ hexo s --debug  #以debug模式运行
```

在终端中显示调试信息并记录到 `debug.log`。当您碰到问题时，可以尝试用调试模式重新执行一次

## 简洁模式

```
$ hexo --silent
```

隐藏终端信息。

# 显示草稿

```
$ hexo --draft
```

显示 `source/_drafts` 文件夹中的草稿文章。

