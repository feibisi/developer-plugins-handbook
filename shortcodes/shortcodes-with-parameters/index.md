---
title: "带参数的短代码"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Shortcodes With Parameters
    - Shortcodes
    - Repos
---

# 带参数的短代码

现在我们已经了解了如何创建[基本短代码](https://developer.wordpress.org/plugins/shortcodes/basic-shortcodes/)以及如何将其用作[自闭合和包裹式短代码](https://developer.wordpress.org/plugins/shortcodes/enclosing-shortcodes/)，接下来我们将探讨如何在短代码 `[$tag]` 和处理函数中使用参数。

短代码 `[$tag]` 可以接受参数，这些参数被称为属性：

```
[wporg title="WordPress.org"]
  享受使用 WordPress.org 短代码的乐趣。
[/wporg]
```

短代码处理函数可以接受 3 个参数：

- `$atts` – 数组 – `[$tag]` 的属性
- `$content` – 字符串 – 短代码内部的内容。在上面的示例中，它将是“享受使用 WordPress.org 短代码的乐趣”。
- `$tag` – 字符串 – `[$tag]` 的名称（即短代码的名称）

```
function wporg_shortcode( $atts = array(), $content = null, $tag = '' ) {}
```

## 解析属性

对用户而言，短代码只是文章内容中带方括号的字符串。用户并不清楚哪些属性可用，也不了解其背后的运行机制。

对插件开发者来说，无法强制规范属性的使用方式。用户可能包含一个属性、两个属性，或完全不包含任何属性。

要掌控短代码的使用方式，可采取以下措施：

- 为处理函数声明默认参数
- 使用 [`array_change_key_case()`](https://www.php.net/manual/en/function.array-change-key-case.php) 对属性数组的键名进行大小写规范化处理
- 通过 [`shortcode_atts()`](https://developer.wordpress.org/reference/functions/shortcode_atts/) 解析属性，需提供默认值数组和用户 `$atts` 参数
- 在返回输出前进行[安全转义处理](https://developer.wordpress.org/apis/security/escaping/)

## Complete Example

Complete example using a basic shortcode structure, taking care of self-closing and enclosing scenarios and securing output.

A `[wporg]` shortcode that will accept a title and will display a box that we can style with CSS.

```
/**
 * The [wporg] shortcode.
 *
 * Accepts a title and will display a box.
 *
 * @param array  $atts    Shortcode attributes. Default empty.
 * @param string $content Shortcode content. Default null.
 * @param string $tag     Shortcode tag (name). Default empty.
 * @return string Shortcode output.
 */
function wporg_shortcode( $atts = [], $content = null, $tag = '' ) {
  // normalize attribute keys, lowercase
  $atts = array_change_key_case( (array) $atts, CASE_LOWER );

  // override default attributes with user attributes
  $wporg_atts = shortcode_atts(
    array(
      'title' => 'WordPress.org',
    ), $atts, $tag
  );

  // start box
  $o = '<div class="wporg-box">';

  // title
  $o .= '<h2>' . esc_html( $wporg_atts['title'] ) . '</h2>';

  // enclosing tags
  if ( ! is_null( $content ) ) {
    // $content here holds everything in between the opening and the closing tags of your shortcode. eg.g [my-shortcode]content[/my-shortcode].
    // Depending on what your shortcode supports, you will parse and append the content to your output in different ways.
    // In this example, we just secure output by executing the_content filter hook on $content.
    $o .= apply_filters( 'the_content', $content );
  }

  // end box
  $o .= '</div>';

  // return output
  return $o;
}

/**
 * Central location to create all shortcodes.
 */
function wporg_shortcodes_init() {
  add_shortcode( 'wporg', 'wporg_shortcode' );
}

add_action( 'init', 'wporg_shortcodes_init' );
```