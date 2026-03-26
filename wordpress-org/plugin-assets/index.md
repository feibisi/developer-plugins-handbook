---
title: "Index"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Plugin Assets
    - Wordpress Org
    - Repos
---

`assets` 文件夹是您插件中用于存放插件展示页所用图像（如插件横幅、图标和截图）的位置。

所有文件都应放置在 SVN 目录的 `assets` 文件夹中，该文件夹适用于插件的所有版本。这是一个顶级目录，与 `trunk` 同级。请**不要**将截图放入 `trunk/assets` 或 `tags/1.0/assets` 中。

所有图像均通过 CDN 提供并经过深度缓存，因此更改或添加后可能需要一些时间才能更新。请在报告图像无法显示前，给予代理服务器一定时间获取并缓存您的图像。通常只需几分钟，但在服务器高负载时（例如 WordPress 主要版本发布前一周及发布期间）可能需要长达 6 小时。

[默认图像尺寸](#default-image-sizes)
-------------------------------------------

图像尺寸应与文件名所暗示的尺寸一致。例如，`banner-772x250.png` 应为 772 像素宽、250 像素高。同样，`icon-256x256.png` 应为 256×256 的正方形。

我们**没有**定义任何新的横幅尺寸，因此请不要自作聪明地重命名您的图像并期望它们能正常显示。它们将无法显示。同样，请勿将图像制作得更大（或更小）却使用现有名称，否则显示效果会很糟糕。

[插件横幅](#plugin-headers)
---------------------------------

插件横幅是您在插件页面顶部看到的图像：

[![](https://i0.wp.com/developer.wordpress.org/files/2015/05/Screenshot-2024-07-02-at-12.50.04%E2%80%AFPM.png?resize=1024%2C472&ssl=1)](https://i0.wp.com/developer.wordpress.org/files/2015/05/Screenshot-2024-07-02-at-12.50.04%E2%80%AFPM.png?ssl=1)

设计横幅图像时，请考虑国际插件目录的使用情况。其中一些目录（如[希伯来语](https://he.wordpress.org/plugins/)和[阿拉伯语](https://ar.wordpress.org/plugins/)）使用从右到左（RTL）的语言。理想情况下，应设计横幅使其包含的元素可以从右到左或从左到右排列。您可以为 RTL 页面创建不同的图像，在文件名中加入 `-rtl`。

### [横幅文件名](#banner-filenames)

* 普通横幅：`banner-772x250.(jpg|png)`
* 本地化普通横幅：`banner-772x250-(rtl|es|es_ES).(jpg|png)`
* 高DPI（视网膜）横幅：`banner-1544x500.(jpg|png)`
* 本地化高DPI（视网膜）横幅：`banner-1544x500-(rtl|es|es_ES).(jpg|png)`

图像可以针对特定语言或所有从右向左（RTL）语言进行本地化。  
区域设置可以指定为完整区域设置（`es_ES`）或部分区域设置（`es`）。如果语言是RTL且未提供特定区域设置的图像，则会检查`rtl`图像。  
请勿在不做修改的情况下将英文图像复制到RTL和特定区域设置文件中，否则将使用英文变体。

关于RTL图像的示例，请查看bbPress的[英文版](https://wordpress.org/plugins/bbpress/)和[阿拉伯语版](https://ar.wordpress.org/plugins/bbpress/)。

关于视网膜图像的示例，请[查看Hello Dolly](https://wordpress.org/extend/plugins/hello-dolly/)或[Pluginception](https://wordpress.org/plugins/pluginception/)。您_不能_单独使用视网膜图像，它仅作为772×250图像的“附加”功能。较大的“视网膜”图像仅用于能够显示更高细节的显示器。

横幅图像的最大尺寸为4MB，但越小越好。

[插件图标](#plugin-icons)
-----------------------------

插件图标是方形图像，在WordPress.org的搜索结果中以及WordPress.org的后台显示在插件旁边。

[![带有其插件图标的Akismet](https://i0.wp.com/developer.wordpress.org/files/2015/05/akismet1.png?resize=651%2C235&ssl=1)](https://i0.wp.com/developer.wordpress.org/files/2015/05/akismet1.png?ssl=1)

除了JPG和PNG格式外，您还可以使用**SVG**。矢量图非常适合图标，因为它们可以缩放到任何尺寸，且文件本身很小。如果您选择使用SVG，则_必须_同时使用PNG图标作为后备，否则您的插件图标在旧版浏览器或Facebook上可能无法正确显示。

如果您不使用图标，系统将为您自动生成一个。示例如下图的圆形图标：

[![自动生成图标示例](https://i0.wp.com/developer.wordpress.org/files/2015/05/auto-generated-icons.jpg?resize=798%2C332&ssl=1)](https://i0.wp.com/developer.wordpress.org/files/2015/05/auto-generated-icons.jpg?ssl=1)

图标的最大文件大小为1MB，但与横幅一样，越小越好。

### [文件名](#filenames)

* 标准：`icon-128x128.(png|jpg|gif)`
* 高分辨率（Retina）：`icon-256x256.(png|jpg|gif)`
* SVG：`icon.svg`

目前没有更改这些尺寸的计划。

[截图](#screenshots)
-------------------

截图会显示在插件的主页上，用于展示插件管理仪表板或实际示例。您的 `readme.txt` 文件中的每一行都应对应**一张**截图。这些行的内容将成为插件页面上截图的说明文字。

例如：`1. 这是一只猴子`

这将在第一张截图下方显示说明文字“这是一只猴子”。推测是关于猴子的截图。

截图**必须**是本地文件才能显示。指向外部文件的链接将无法使用。

截图的最大文件大小为 10MB，但一如既往，文件越小越好。

### [文件名](#filenames-2)

* `screenshot-1.(png|jpg)`
* `screenshot-2.(png|jpg)`

所有文件名应为小写；大写名称将无法生效。

截图可以[类似横幅](#banner-filenames)进行本地化，当插件在德语环境下查看时，以下文件名将优先于上述英文名称：

* `screenshot-1-de.(png|jpg)`
* `screenshot-2-de.(png|jpg)`

[问题](#issues)
-----------------

如果您发现用户从 WordPress.org 插件目录页面点击您的图片时，图片会直接下载，您需要通过 SVN 更改上传方式。这是由于某些图片以 `Content-Type application/octet-stream` 格式发送所致。

要修复此问题，请运行以下命令：

    svn propset svn:mime-type image/png *.png
    svn propset svn:mime-type image/jpeg *.jpg

或者，插件作者可以在其 ~/.subversion/config 文件中设置：

    *.png = svn:mime-type=image/png
    *.jpg = svn:mime-type=image/jpeg

但这仅适用于新文件。修复已提交的文件仍需使用上述命令。