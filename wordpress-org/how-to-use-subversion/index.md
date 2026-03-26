---
title: "使用 Subversion"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - How To Use Subversion
    - Wordpress Org
    - Repos
---

# 使用 Subversion

SVN（即 Subversion）是一种类似于 Git 的版本控制系统。您可以通过命令行或多种图形界面应用程序（如 [Tortoise SVN](https://tortoisesvn.net/)、[SmartSVN](https://www.smartsvn.com/) 等）来使用它。如果您是 SVN 新手，建议先查阅 [SVN 客户端对比](https://en.wikipedia.org/wiki/Comparison_of_Subversion_clients)再选择最适合您的工具。

本文档并非完整详尽的 SVN 使用指南，而是针对 WordPress.org 插件开发的快速入门指引。如需更全面的文档，请参阅 [《SVN 手册》](https://svnbook.red-bean.com/)。

这里我们将介绍与 WordPress.org 托管相关的 SVN 基础操作。SVN 及几乎所有代码仓库服务的基本概念都是相通的。

更多信息请查阅以下文档：

- [readme.txt 文件使用说明](https://developer.wordpress.org/plugins/wordpress-org/how-your-readme-txt-works/)
- [插件资源文件（横幅图片与图标）使用指南](https://developer.wordpress.org/plugins/wordpress-org/plugin-assets/)

[warning]SVN 与插件目录是**发布**仓库。不同于 Git，您不应提交每个微小改动，否则可能影响性能。请仅将**已完成**的更改推送到 SVN 仓库。[/warning]

## 概述

您的所有文件都将集中存储在我们服务器上的 **SVN 代码库**中。任何人都可以从该代码库**检出**您的插件文件副本到本地计算机，但作为插件作者，只有您拥有**签入**权限。这意味着您可以在本地计算机上修改文件、添加新文件、删除文件，并将这些更改上传回中央服务器。正是通过这个签入过程，既更新了代码库中的文件，也更新了 WordPress.org 插件目录中显示的信息。

Subversion 会跟踪所有这些更改，以便您在需要时可以随时回溯查看旧版本或**修订版**。除了记录每个单独的修订版，您还可以指示 Subversion 为代码库的特定修订版**打标签**以便于引用。标签非常适合[标记插件的不同发布版本](https://developer.wordpress.org/plugins/wordpress-org/how-to-use-subversion/#tags)，并且是确保 WordPress.org 上显示正确版本并为用户更新的唯一完全受支持的方法。

## 您的账户

您在 SVN 中的账户将使用您提交插件时所使用的账户用户名（而非邮箱）。这也是您在 WordPress 论坛中使用的用户 ID。

请注意，_大小写敏感_ —— 如果您的用户名是 JaneDoe，则必须使用大写的 J 和 D，否则 SVN 将无法识别。您可以在[个人资料页面](https://profiles.wordpress.org/me/profile/edit/group/1/)查看您用户名的具体大小写格式。

如需重置密码，请访问 [login.wordpress.org](https://login.wordpress.org/lostpassword)。

## SVN 目录

所有 SVN 仓库默认会创建三个目录。

```
/assets/
/tags/
/trunk/
```

- `assets` 用于存放[截图、插件头部信息和插件图标](https://developer.wordpress.org/plugins/wordpress-org/plugin-assets/)。
- 开发工作应放在 `trunk` 中。
- 发布版本应置于 `tags` 内。

_过去用于存放分支代码的 `/branches/` 目录因未被使用，现已不再默认创建。_

### 主干目录

[warning]请勿将你的 _主_ 插件文件放在 trunk 的子文件夹中，例如 `/trunk/my-plugin/my-plugin.php`，否则会破坏下载功能。你可以使用子文件夹来存放包含文件。[/warning]

`/trunk` 目录是你的插件代码应该存放的位置。主干目录可以被视为最新、最优秀的代码，但这不一定是最近的 _稳定_ 代码。主干目录用于开发版本。理想情况下，主干目录中的代码应该始终是可运行的，但由于它不一定是“稳定”版本，偶尔可能会存在错误。对于简单的插件，主干目录可能是代码的唯一版本，这也是完全可以接受的。

即使你在其他地方（如 git 仓库）进行开发工作，我们建议你保持 trunk 文件夹与你的代码同步更新，以便于进行 SVN 比较。

### 标签

`/tags` 目录用于存放插件的各个版本。此处的子目录版本号应与插件版本号保持一致。务必使用标签文件夹和正确的版本控制，以确保用户获取正确的代码。

插件 1.0 版本应位于 `/tags/1.0`，1.1 版本位于 `/tags/1.1`，依此类推。

我们**强烈**建议采用[语义化软件版本控制](https://semver.org/)。

### 资源文件

[info]另请参阅：[插件资源文件工作原理](https://developer.wordpress.org/plugins/wordpress-org/plugin-assets/)[/info]

资源文件目录存放您的截图、标题图像和插件图标。目录中部分旧版插件可能仍将截图文件置于 /trunk 目录，但这并非推荐做法。所有新插件都应将其截图置于 /assets 目录。这有助于保持插件文件体积小巧，因为无需随插件本身向 WordPress 安装环境发送截图文件。

### 分支

[alert]`/branches/` 目录默认不再创建，因为它基本未被使用。此部分可视为已弃用，仅作参考用途。[/alert]

`/branches` 目录可用于存储插件的分支。例如开发中的版本或测试代码等。

WordPress.org 系统**完全不会**使用分支目录，它严格被视为供开发者按需使用的空间。由于默认不再创建，若您不再需要，可忽略此目录。

## 最佳实践

为了让其他开发者更容易理解您的代码，以下实践被认为是最佳方案。

### 开发时请勿使用 SVN

这常常令人困惑。与 GitHub 不同，SVN 是一个 _发布_ 系统，而非开发系统。您无需提交和推送每一个微小的更改，实际上这样做对系统有害。每次将代码推送到 SVN 时，它都会为 SVN 中的所有版本重新构建 _所有_ 的 zip 文件。这就是为什么有时您的插件更新需要长达 6 小时才会显示。相反，您应该在准备就绪时一次性推送。

### 将代码存放于主干文件夹

许多人将 `trunk` 用作占位符。虽然可以直接更新 `trunk` 中的 `readme.txt` 文件并将所有内容放入标签文件夹，但这样做会使代码变更对比更加困难。相反，主干应包含代码的最新版本，即使该版本是测试版。

### 始终标记版本发布

虽然可以将主干分支用作插件的稳定标签，但这实际上不受支持也不推荐。相反，版本应正确标记并迭代发布。这将确保与任何自动更新器的完全兼容性，并在代码出现问题时允许回滚。

### 从主干创建标签

与其直接将代码推送到标签文件夹，你应该先在主干中编辑代码，并在 readme 中完善稳定版本信息，*然后*再将代码从主干复制到新标签。

这不仅便于查看任何更改，你还能提交更小的变更集，因为 SVN 只会更新已修改的代码。这将节省你的时间并减少潜在错误（例如更新到错误的稳定标签或将问题代码推送给用户）。

不必担心标签文件夹短暂不存在的情况。你可以使用 `svn cp` 将主干复制到标签，然后同时将它们推送到 SVN。

如果你在本地操作，则可以一次性更新主干并从中创建标签。检出仓库根目录，更新 /trunk 中的文件，然后执行 `svn copy /trunk /tags/1.2.3`（或任意版本号），最后一次性提交所有更改。SVN 是基于差异的系统，只要你使用 svn 执行复制操作，它就会保留历史记录，让他人轻松跟进。

### 删除旧版本

由于 SVN 是一个发布仓库，许多开发者选择删除其插件的旧版（不再支持的）版本。截至 2019 年，这不再能加速发布，因为构建过程仅处理已更改文件的标签。

## 示例

### 开始一个新插件

要启动你的插件，你需要将已有的文件添加到新的 SVN 仓库中。

首先在你的机器上创建一个本地目录，用于存放 SVN 仓库的副本：

```
mkdir my-local-dir
```

接下来，检出预构建的仓库

```
svn co https://plugins.svn.wordpress.org/your-plugin-name my-local-dir
> A my-local-dir/trunk
> A my-local-dir/branches
> A my-local-dir/tags
> Checked out revision 11325.
```

在我们的示例中，Subversion 已将中央 SVN 仓库中的所有目录添加（"A" 表示 "add"）到你的本地副本中。

要添加你的代码，请导航到 `my-local-dir` 文件夹：

```
cd my-local-dir
```

现在，你可以通过命令行的复制/粘贴命令，或拖放操作，将你的文件添加到仓库本地副本的 `trunk/` 目录中。使用你习惯的任何方式。

[warning]不要将你的 _主_ 插件文件放在 trunk 的子文件夹中，例如 `/trunk/my-plugin/my-plugin.php`，因为这会破坏下载。你可以为包含的文件使用子文件夹。[/warning]

一旦你的文件放入 trunk 文件夹，你必须让 Subversion 知道你想将这些新文件添加回中央仓库。

```
cd my-local-dir
svn add trunk/*
> A trunk/my-plugin.php
> A trunk/readme.txt
```

添加完所有文件后，你将把更改签入回中央仓库。

```
svn ci -m 'Adding first version of my plugin'
> Adding trunk/my-plugin.php
> Adding trunk/readme.txt
> Transmitting file data .
> Committed revision 11326.
```

所有签入都必须包含提交信息。

如果提交因“访问被禁止”而失败，并且你**确定**你有提交权限，请在签入命令中添加你的用户名和密码。

```
svn ci -m 'Adding first version of my plugin' --username your_username --password your_password
```

请记住，你的用户名是 _区分大小写_ 的。

### 编辑现有文件

当你的插件位于目录中后，很可能需要在某个时候编辑代码。

首先进入你的本地仓库副本，并确保它是最新的。

```
cd my-local-dir/
svn up
> At revision 11326.
```

在上面的例子中，我们完全是最新的。如果中央仓库有更改，它们会被下载并合并到你的本地副本中。

现在你可以使用你喜欢的任何编辑器来编辑需要更改的文件。

如果你没有使用 SVN 图形界面工具（如 SubVersion 或 Coda），你仍然可以在更改后检查并查看本地副本与中央仓库之间的差异。首先我们检查本地副本的状态：

```
svn stat
> M trunk/my-plugin.php
```

这告诉我们本地的 `trunk/my-plugin.php` 与我们从中央仓库下载的副本不同（"M" 表示 "已修改"）。

让我们看看该文件中具体发生了什么变化，以便我们可以检查并确保一切看起来正确。

```
svn diff
> * 输出的内容本质上是你的本地副本与你下载的原始副本之间
  * 执行标准 `diff -u` 命令的结果。
```

如果一切看起来都没问题，那么是时候将这些更改签入中央仓库了。

```
svn ci -m "fancy new feature: now you can foo *and* bar at the same time"
> Sending trunk/my-plugin.php
> Transmitting file data .
> Committed revision 11327.
```

现在你已经成功更新了主干。

### 为新版本“打标签”

每次正式发布插件时，都应该为该版本的代码副本打上标签。这能让您的用户轻松获取最新（或旧版）版本，让您更轻松地跟踪变更，并让 WordPress.org 插件目录知道应建议用户下载哪个版本的插件。

首先将代码复制到 `tags/` 目录的子目录中。为了适配 WordPress.org 插件浏览器，新的子目录应始终采用版本号格式。例如 `2.0.1.3` 是**正确**的，而 `Cool hotness tag` 是**错误**的。

我们希望使用 `svn cp` 而非普通的 `cp` 命令，以利用 SVN 的功能特性。

```
svn cp trunk tags/2.0
> A tags/2.0
```

与往常一样，提交变更。

```
svn ci -m "tagging version 2.0"
> Adding tags/2.0
> Adding tags/2.0/my-plugin.php
> Adding tags/2.0/readme.txt
> Committed revision 11328.
```

为新版本打标签时，**请务必更新** [`trunk/readme.txt`](https://wordpress.org/plugins/developers/#readme) 中的 `Stable Tag` 字段为新版本号。

恭喜！您已成功更新代码！

## 注意事项

切勿将任何你不愿或未准备好部署给所有插件用户的内容放入 SVN。这**包括**供应商文件、`.gitignore` 以及其他所有内容。

同时，请永远不要上传 zip 压缩包。与大多数代码仓库系统一样，SVN 要求你上传独立的文件。

### See Also

- [How the readme.txt works](https://developer.wordpress.org/plugins/wordpress-org/how-your-readme-txt-works/).
- [How plugin assets (header images and icons) work](https://developer.wordpress.org/plugins/wordpress-org/plugin-assets/).
- [The SVN Book](https://svnbook.red-bean.com/).