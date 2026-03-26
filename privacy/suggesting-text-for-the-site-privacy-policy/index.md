---
title: "Index"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Suggesting Text For The Site Privacy Policy
    - Privacy
    - Repos
---

为网站隐私政策提供建议文本
-------------------------------------------

每个收集、使用或存储用户数据，或将其传递给外部来源或第三方的插件，都应在隐私政策文本框中添加建议文本部分。最佳实现方式是使用 `wp_add_privacy_policy_content( $plugin_name, $policy_text )`。这将允许网站管理员将这些信息整合到其网站的隐私政策中。

为方便用户使用，建议文本应涵盖默认隐私政策中提出的问题：

- 我们收集哪些个人数据及收集原因
	- 用户自行输入的信息
	- WP：联系表单
	- WP：评论
	- WP：Cookie
	- WP：第三方嵌入内容
	- 分析数据
- 我们与谁共享您的数据
- 我们保留您的数据多长时间
- 您对数据拥有哪些权利
- 我们将数据发送至何处
- 您的联系信息
- 我们如何保护您的数据
- 我们有哪些数据泄露处理程序
- 我们从哪些第三方接收数据
- 我们如何使用用户数据进行自动化决策和/或画像分析
- 任何行业监管披露要求

虽然并非所有问题都适用于所有插件，但我们建议特别关注数据共享相关部分。

## 代码示例

[info]建议在 [admin_init](https://developer.wordpress.org/reference/hooks/admin_init/) 操作期间调用 [`wp_add_privacy_policy_content()`](https://developer.wordpress.org/reference/functions/wp_add_privacy_policy_content/)。在操作钩子外调用可能导致问题，详情参见工单 [#44142](https://core.trac.wordpress.org/ticket/44142)。[/info]

[info]可通过专用 `.privacy-policy-tutorial` CSS 类提供补充信息。当复制章节内容时，应用了此 CSS 类的 HTML 元素内的所有内容都将从剪贴板中省略。[/info]

```
/**
 * 添加隐私政策声明。
 */
function wporg_add_privacy_policy_content() {
  if ( ! function_exists( 'wp_add_privacy_policy_content' ) ) {
    return;
  }
  $content = '<p class="privacy-policy-tutorial">' . __( '建议文本的引导性内容。', 'text-domain' ) . '</p>'
  . '<strong class="privacy-policy-tutorial">' . __( '建议文本：', 'my_plugin_textdomain' ) . '</strong> '
  . sprintf(
    __( '当您在本站留言时，我们会将您的姓名、邮箱地址、IP地址和评论文本发送至 example.com。Example.com 不会保留您的个人数据。Example.com 的隐私政策请访问<a href="%1$s" target="_blank">此处</a>。', 'text-domain' ),
    'https://example.com/privacy-policy'
  );
  wp_add_privacy_policy_content( 'Example Plugin', wp_kses_post( wpautop( $content, false ) ) );
}

add_action( 'admin_init', 'wporg_add_privacy_policy_content' );
```