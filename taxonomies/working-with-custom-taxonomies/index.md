---
title: "使用自定义分类法"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Working With Custom Taxonomies
    - Taxonomies
    - Repos
---

# 使用自定义分类法

## 分类法简介

要了解什么是分类法及其功能，请阅读[分类法](https://developer.wordpress.org/plugins/taxonomies/)介绍。

## 自定义分类法

就分类系统而言，“分类目录”和“标签”的结构性不强，因此开发者创建自己的分类法可能更为有利。

WordPress 允许开发者创建**自定义分类法**。当需要创建独特的命名系统，并以可预测的方式在后台访问它们时，自定义分类法非常有用。

## 为何使用自定义分类法？

你可能会问："既然可以用分类和标签来组织内容，为什么还要费心创建自定义分类法呢？"

嗯……让我们举个例子。假设我们有一位客户是厨师，她想让你创建一个展示原创菜谱的网站。

一种组织网站的方式是：创建一个名为"菜谱"的自定义文章类型来存储她的食谱，然后创建一个"餐点类型"分类法来区分"开胃菜"和"甜点"，最后再创建一个"食材"分类法来区分"鸡肉"和"巧克力"类菜肴。

这些分组*原本*可以用分类或标签来定义——你可以创建一个"餐点类型"分类，下设"开胃菜"和"甜点"子分类；再创建一个"食材"分类，为每种食材设置子分类。

使用自定义分类法的主要优势在于：你可以独立于常规分类和标签来引用"餐点类型"和"食材"。它们甚至在管理后台拥有独立的菜单。

此外，创建自定义分类法还能让你构建定制化界面，这将简化客户的操作流程，并使数据录入过程更符合其业务特性。

现在想象一下：如果这些自定义分类法和界面被实现在一个插件中——你就构建了一个可用于任何WordPress网站的菜谱插件。

## 示例：课程分类法

以下示例将展示如何创建一个插件，为默认的 `post` 文章类型添加自定义分类法“课程”。请注意，添加自定义分类法的代码不必在独立的插件中；如果需要，它可以包含在主题中或作为现有插件的一部分。

在尝试创建自己的插件之前，请务必阅读 [插件基础](https://developer.wordpress.org/plugins/plugin-basics/) 章节。

### 步骤 1：准备工作

前往 **文章 > 写文章** 页面。您会注意到目前只有“分类目录”和“标签”。

![尚无自定义分类法元框](https://make.wordpress.org/docs/files/2014/02/no-custom-taxonomy-meta-box.png)

## Step 2: Creating a New Plugin

Register the Taxonomy "course" for the post type "post" using the `init` action hook.

```
/*
 * Plugin Name: Course Taxonomy
 * Description: A short example showing how to add a taxonomy called Course.
 * Version: 1.0
 * Author: developer.wordpress.org
 * Author URI: https://codex.wordpress.org/User:Aternus
 */

function wporg_register_taxonomy_course() {
  $labels = array(
    'name'              => _x( 'Courses', 'taxonomy general name' ),
    'singular_name'     => _x( 'Course', 'taxonomy singular name' ),
    'search_items'      => __( 'Search Courses' ),
    'all_items'         => __( 'All Courses' ),
    'parent_item'       => __( 'Parent Course' ),
    'parent_item_colon' => __( 'Parent Course:' ),
    'edit_item'         => __( 'Edit Course' ),
    'update_item'       => __( 'Update Course' ),
    'add_new_item'      => __( 'Add New Course' ),
    'new_item_name'     => __( 'New Course Name' ),
    'menu_name'         => __( 'Course' ),
  );
  $args   = array(
    'hierarchical'      => true, // make it hierarchical (like categories)
    'labels'            => $labels,
    'show_ui'           => true,
    'show_admin_column' => true,
    'query_var'         => true,
    'rewrite'           => [ 'slug' => 'course' ],
  );
  register_taxonomy( 'course', [ 'post' ], $args );
}
add_action( 'init', 'wporg_register_taxonomy_course' );
```

### 步骤 3：查看结果

激活你的插件，然后转到 **文章 > 写文章**。你应该会看到一个用于“课程”分类法的新元框。

![课程分类法文章界面](https://make.wordpress.org/docs/files/2014/02/courses_taxonomy_post_screen.png)

### 代码解析

以下讨论将分解上述代码，描述其功能与参数。

函数 `wporg_register_taxonomy_course` 包含了注册自定义分类法所需的所有步骤。

`$labels` 数组存储了自定义分类法的标签。

这些标签将用于在管理区域显示关于该分类法的各种信息。

`$args` 数组存储了创建自定义分类法时将使用的配置选项。

函数 [`register_taxonomy()`](https://developer.wordpress.org/reference/functions/register_taxonomy/) 使用 `$args` 数组进行配置，为 `post` 文章类型创建了一个标识符为 `course` 的新分类法。

函数 [`add_action()`](https://developer.wordpress.org/reference/functions/add_action/) 将 `wporg_register_taxonomy_course` 函数的执行绑定到 `init` 动作钩子上。

#### $args

$args 数组保存了自定义分类法的重要配置，它指示 WordPress 该分类法应如何工作。

### 总结

查看 [`register_taxonomy()`](https://developer.wordpress.org/reference/functions/register_taxonomy/) 函数以获取完整参数列表及各参数作用说明。

以我们的课程分类法为例，WordPress 将自动为 `course` 分类法创建归档页面和子页面。

归档页面位于 `/course/`，其下将根据术语别名生成子页面（`/course/%%term-slug%%/`）。

## 使用您的分类法

WordPress 提供了**许多**用于与您的自定义分类法及其内部术语进行交互的函数。

以下是一些示例：

- `the_terms`：接受一个分类法参数，并以列表形式渲染术语。
- `wp_tag_cloud`：接受一个分类法参数，并渲染术语的标签云。
- `is_taxonomy`：允许您判断给定的分类法是否存在。