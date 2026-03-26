---
title: "TinyMCE 增强型短代码"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Tinymce Enhanced Shortcodes
    - Shortcodes
    - Repos
---

# TinyMCE 增强型短代码

可以在 TinyMCE 可视化编辑器中解析短代码，使其渲染实际内容而非短代码本身。

切换到 `文本` 标签页可再次查看实际短代码。

以下是使用此功能的 WordPress 内置短代码。

## 音频短代码

`[audio]` 短代码允许嵌入单个音频文件。

![音频短代码](https://i3.wp.com/developer.wordpress.org/files/2014/09/shortcodes-tinymce-enhanced-shortcodes-01.png)

## 标题短代码

`[caption]` 短代码将图像包裹在 div 中，并在标题周围放置 `<p class="wp-caption-text">` 标签。

![标题短代码](https://i3.wp.com/developer.wordpress.org/files/2014/09/shortcodes-tinymce-enhanced-shortcodes-02.png)

## 图库短代码

`[gallery]` 短代码允许在 div 中一次性嵌入多张图像。

![图库短代码](https://i3.wp.com/developer.wordpress.org/files/2014/09/shortcodes-tinymce-enhanced-shortcodes-03.png)

## 播放列表短代码

`[playlist]` 短代码允许附加多个媒体文件，并通过 html5 播放列表进行渲染。

![播放列表短代码](https://i3.wp.com/developer.wordpress.org/files/2014/09/shortcodes-tinymce-enhanced-shortcodes-04.png)

## 视频短代码

`[video]` 短代码与 `[audio]` 短代码非常相似；它只是渲染视频而非音频。

![视频短代码](https://i3.wp.com/developer.wordpress.org/files/2014/09/shortcodes-tinymce-enhanced-shortcodes-05.png)