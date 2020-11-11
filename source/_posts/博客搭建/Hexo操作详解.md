---
title: Hexo文章操作详解
date: 2019-10-04 10:58:51
tags: 
  - 博客搭建
  - Hexo
categories:
  - 博客搭建
---

[TOC]

# 网站配置

网站根目录下的 `_config.yml` 中修改大部分的配置

## 网站

| 参数          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `title`       | 网站标题                                                     |
| `subtitle`    | 网站副标题                                                   |
| `description` | 网站描述                                                     |
| `keywords`    | 网站的关键词。使用半角逗号 `,` 分隔多个关键词。              |
| `author`      | 您的名字                                                     |
| `language`    | 网站使用的语言                                               |
| `timezone`    | 网站时区。Hexo 默认使用您电脑的时区。[时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。比如说：`America/New_York`, `Japan`, 和 `UTC` 。 |

其中，`description`主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author`参数用于主题显示文章的作者。

## 网址

| 参数                 | 描述                                                         | 默认值                      |
| :------------------- | :----------------------------------------------------------- | :-------------------------- |
| `url`                | 网址                                                         |                             |
| `root`               | 网站根目录                                                   |                             |
| `permalink`          | 文章的 [永久链接](https://hexo.io/zh-cn/docs/permalinks) 格式 | `:year/:month/:day/:title/` |
| `permalink_defaults` | 永久链接中各部分的默认值                                     |                             |

如果你的网站存放在子目录中，例如 `http://yoursite.com/blog`，则请将您的 `url` 设为 `http://yoursite.com/blog` 并把 `root` 设为 `/blog/`

## 文章

| 参数                | 描述                                 | 默认值    |
| :------------------ | :----------------------------------- | :-------- |
| `new_post_name`     | 新文章的文件名称                     | :title.md |
| `default_layout`    | 预设布局                             | post      |
| `auto_spacing`      | 在中文和英文之间加入空格             | false     |
| `titlecase`         | 把标题转换为 title case              | false     |
| `external_link`     | 在新标签中打开链接                   | true      |
| `filename_case`     | 把文件名称转换为 (1) 小写或 (2) 大写 | 0         |
| `render_drafts`     | 显示草稿                             | false     |
| `post_asset_folder` | 启动 Asset 文件夹                    | false     |
| `relative_link`     | 把链接改为与根目录的相对位址         | false     |
| `future`            | 显示未来的文章                       | true      |
| `highlight`         | 代码块的设置                         |           |

## 分类 & 标签

| 参数               | 描述     | 默认值          |
| :----------------- | :------- | :-------------- |
| `default_category` | 默认分类 | `uncategorized` |
| `category_map`     | 分类别名 |                 |
| `tag_map`          | 标签别名 |                 |

## 日期 / 时间格式

Hexo 使用 [Moment.js](http://momentjs.com/) 来解析和显示时间。

| 参数          | 描述     | 默认值       |
| :------------ | :------- | :----------- |
| `date_format` | 日期格式 | `YYYY-MM-DD` |
| `time_format` | 时间格式 | `HH:mm:ss`   |

## 分页

| 参数             | 描述                                | 默认值 |
| :--------------- | :---------------------------------- | :----- |
| `per_page`       | 每页显示的文章量 (0 = 关闭分页功能) | `10`   |
| `pagination_dir` | 分页目录                            | `page` |

## 扩展

| 参数             | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| `theme`          | 当前主题名称。值为`false`时禁用主题                          |
| `theme_config`   | 主题的配置文件。在这里放置的配置会覆盖主题目录下的 _config.yml 中的配置 |
| `deploy`         | 部署部分的设置                                               |
| `meta_generator` | [Meta generator](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta#属性) 标签。 值为 `false` 时 Hexo 不会在头部插入该标签 |

# 主题配置



# 文章文件

## 创建文章

```shell
hexo new [文章布局][参数] "文章标题"  #文章布局可省略，默认是post
```

执行完成后可以在` /source/_posts` 下看到一个“`文章标题.md`”的文章文件。是 `Markdown` 格式的文件，头部为布局文件中的形式。最好不要直接在` /source/_posts`下直接创建`.md`文件。如果要直接创建则要加上头部信息(Front-matter)

参数：

| 参数              | 描述                                          |
| :---------------- | :-------------------------------------------- |
| `-p`, `--path`    | 自定义新文章的路径                            |
| `-r`, `--replace` | 如果存在同名文章，将其替换                    |
| `-s`, `--slug`    | 文章的 Slug，作为新文章的文件名和发布后的 URL |

默认情况下，Hexo 会使用文章布局来决定文章文件的路径。对于page布局来说，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 `index.md` 文件。可以使用 `--path` 参数来覆盖上述行为、自行决定文件的目录：

```
hexo new page --path about/me "About me"
```

以上命令会创建一个 `source/about/me.md` 文件，同时 Front Matter 中的 title 为 `"About me"`

## 文章布局（Layout）

在新建文章时，Hexo 会根据 `scaffolds` 文件夹内相对应的文章布局文件来建立`.md`文件。文章的布局（layout），默认为 `post`（对应`scaffolds`中的`post.md`文件），可以通过修改 `_config.yml` 中的 `default_layout` 参数来指定默认布局

例如：

```
hexo new photo "My Gallery"
```

在执行这行指令时，Hexo 会尝试在 `scaffolds` 文件夹中寻找 `photo.md`，并根据其内容建立`"My Gallery.md"`文章



### 默认布局：`post`、`page`、`draft`

在创建三种不同类型的文件时，它们将会被保存到不同的路径；自定义的其他布局和 `post` 相同，都将储存到 `source/_posts` 文件夹。

| 布局    | 路径             |
| :------ | :--------------- |
| `post`  | `source/_posts`  |
| `page`  | `source`         |
| `draft` | `source/_drafts` |

草稿：

draft是Hexo 的一种特殊布局，这种布局在建立时会被保存到 `source/_drafts` 文件夹，默认不渲染，可通过 `publish` 命令将草稿移动到 `source/_posts` 文件夹进行渲染，该命令的使用方式与 `new` 类似

```
hexo publish [文章布局] "文章标题"  #文章布局可省略，默认是post
```

草稿默认不会显示在页面中，可在执行时加上 `--draft` 参数，或是把 `render_drafts` 参数设为 `true` 来预览草稿。

### 自定义布局

支持自定义布局，可模仿`scaffolds/post.md`文件书写，详细见 Front-matter



## 文件名称

Hexo 默认以标题做为文件名称，可编辑站点配置文件的 `new_post_name` 参数来改变默认的文件名称，推荐默认设置 `:year-:month-:day-:title.md` 可更方便的通过日期来管理文章。

| 变量       | 描述                                |
| :--------- | :---------------------------------- |
| `:title`   | 标题（小写，空格将会被替换为短杠）  |
| `:year`    | 建立的年份，比如， `2015`           |
| `:month`   | 建立的月份（有前导零），比如， `04` |
| `:i_month` | 建立的月份（无前导零），比如， `4`  |
| `:day`     | 建立的日期（有前导零），比如， `07` |
| `:i_day`   | 建立的日期（无前导零），比如， `7`  |

## 文章头部（Front-matter）

Front-matter 是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

一般Front-matter：

```markdown
title: your title
date: year-month-day
tags:
   - your tag1
   - your tag2
categories:
   - your category1
   - your category2
```

### 参数

以下是预先定义的参数，可在模板中使用这些参数值并加以利用。

| 参数         | 描述                                                 | 默认值       |
| :----------- | :--------------------------------------------------- | :----------- |
| `layout`     | 布局                                                 |              |
| `title`      | 标题                                                 |              |
| `date`       | 建立日期                                             | 文件建立日期 |
| `updated`    | 更新日期                                             | 文件更新日期 |
| `comments`   | 开启文章的评论功能                                   | true         |
| `tags`       | 标签（不适用于分页）                                 |              |
| `categories` | 分类（不适用于分页）                                 |              |
| `permalink`  | 覆盖文章网址                                         |              |
| `keywords`   | 仅用于 meta 标签和 Open Graph 的关键词（不推荐使用） |              |

### 分类和标签

只有文章支持分类和标签，可在 Front-matter 中设置。分类具有顺序性和层次性，也就是说 `Foo, Bar` 不等于 `Bar, Foo`；而标签没有顺序和层次。

```
categories:
 - Diary
tags:
 - PS3
 - Games
```

# 文章资源

资源（Asset）指`source` 文件夹中除了文章以外的所有文件，例如图片、CSS、JS 文件等。使用文章资源的方式有两种：使用Markdow语法或使用插件

## 使用Markdown语法和相对路径来引用资源

如果你的Hexo项目中只有少量图片，最简单的方法就是将它们放在 `source/images` 文件夹中。然后使用Markdown语法 `![](/images/image.jpg)` 的方法访问它们。

但通过常规的 markdown 语法和相对路径来引用图片和其它资源可能会导致它们在存档页或者主页上显示不正确

## 使用文章资源文件夹和相对路径引用的标签插件

1. 打开文件资源文件夹

   Hexo提供了更组织化的方式来管理资源。通过将 主站配置文件`_config.yml` 文件中的 `post_asset_folder` 选项设为 `true` 来打开。

   ```
   _config.ymlpost_asset_folder: true
   ```

   当资源文件管理功能打开后，Hexo将会在你每一次通过 `hexo new [layout] <title>` 命令创建新文章时自动创建一个文件夹。这个资源文件夹将会有与这个文章文件一样的名字。将所有与你的文章有关的资源放在这个关联文件夹中之后，可以通过相对路径来引用它们，这样就得到了一个更简单而且方便得多的工作流。

2. 使用相对路径引用的标签插件插入图片

   Hexo 3 这种标签插件已被加入到了核心代码中。这使得可以更简单地在文章中引用你的资源，语句如下：

   ```
   {% asset_path slug %}
   {% asset_img slug [title] %}
   {% asset_link slug [title] %}
   ```

   例如：当你打开文章资源文件夹功能后，把一个 `example.jpg` 图片放在了对应的资源文件夹中

   ```
   {% asset_img example.jpg This is an example image %}
   ```

​	通过这种方式，图片将会同时出现在文章和主页以及归档页中。

# 插件

