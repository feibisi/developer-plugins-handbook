---
title: "将 WP-Cron 接入系统任务调度器"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Hooking Wp Cron Into The System Task Scheduler
    - Cron
    - Repos
---

# 将 WP-Cron 接入系统任务调度器

如前所述，WP-Cron 不会持续运行，这对于需要准时执行的关键任务可能造成问题。对此有一个简单的解决方案：只需将系统任务调度器设置为按所需间隔（或在特定时间）运行。最简单的方案是使用工具向 `wp-cron.php` 文件发起网络请求。

在系统中设置好任务后，还需完成一个步骤。WordPress 仍会在每次页面加载时运行 WP-Cron，这已不再必要且会增加服务器资源消耗。可通过 `wp-config.php` 文件禁用 WP-Cron。打开 `wp-config.php` 文件进行编辑，添加以下行：

```
define( 'DISABLE_WP_CRON', true );
```

## Windows 系统

Windows 将其基于时间的调度系统称为"任务计划程序"，可通过控制面板的**管理工具**访问。

具体设置方法因服务器配置而异。一种方法是使用 PowerShell 和基本任务。创建基本任务后，可使用以下命令调用 WordPress Cron 脚本：

```
powershell "Invoke-WebRequest http://YOUR_SITE_URL/wp-cron.php"
```

## MacOS 与 Linux 系统

Mac OS X 和 Linux 均使用 cron 作为时间调度系统，通常通过终端执行 `crontab -e` 命令访问。需注意任务将根据执行命令的系统用户身份，以普通用户或 root 权限运行。

Cron 有特定的语法格式，包含以下部分：

- 分钟
- 小时
- 月份中的日期
- 月份
- 星期中的日期
- 要执行的命令

![Cron 调度示意图](https://i3.wp.com/developer.wordpress.org/files/2014/10/plugin-wp-cron-cron-scheduling.png)

若某个时间字段无需限制，应使用星号（`*`）。例如要每 15 分钟执行命令（忽略具体小时、日期或月份），可配置为：

多数服务器已安装 `wget` 工具，这是调用 WordPress Cron 脚本的便捷方式：

```
wget --delete-after https://example.com/wp-cron.php
```

[info]若未使用 –delete-after 参数，wget 将保存 HTTP GET 请求的输出结果。[/info]

每日午夜触发站点 WordPress Cron 的配置示例如下：

```
0 0 * * * wget --delete-after https://example.com/wp-cron.php
```