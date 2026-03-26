---
title: "检查用户权限"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Checking User Capabilities
    - Security
    - Repos
---

# 检查用户权限

如果你的插件允许用户提交数据——无论是在管理端还是公共端——都应该检查用户权限。

## 用户角色与权限

创建高效安全层的最重要步骤是建立用户权限系统。WordPress 通过[用户角色与权限](https://developer.wordpress.org/plugins/users/roles-and-capabilities/)的形式提供了这一功能。

每个登录 WordPress 的用户都会根据其用户角色自动分配特定的用户权限。

**用户角色**只是表示用户所属组的另一种说法。每个组都有一组特定的预定义权限。

例如，你网站的主要用户将拥有管理员角色，而其他用户可能拥有编辑或作者等角色。一个角色可以分配给多个用户，即一个网站可能有两个管理员。

**用户权限**是你分配给每个用户或用户角色的具体许可。

例如，管理员拥有 `manage_options` 权限，这允许他们查看、编辑和保存网站选项。而编辑则缺乏此权限，这将阻止他们操作选项。

这些权限会在管理界面的各个节点进行检查。根据分配给角色的权限，WordPress 体验中的菜单、功能和其他方面可能会被添加或移除。

**在构建插件时，请确保仅在当前用户拥有必要权限时才运行你的代码。**

### 层级结构

用户角色越高，拥有的权限就越多。每个用户角色都会继承层级结构中前序角色的所有权限。

例如，在单站点安装中最高级别的用户角色"管理员"，会继承以下角色及其权限："订阅者"、"投稿人"、"作者"和"编辑"。

## Examples

### No Restrictions

The example below creates a link on the frontend which gives the ability to trash posts. Because this code does not check user capabilities, **it allows any visitor to the site to trash posts!**

```
/**
 * Generate a Delete link based on the homepage url.
 *
 * @param string $content   Existing content.
 *
 * @return string|null
 */
function wporg_generate_delete_link( $content ) {
  // Run only for single post page.
  if ( is_single() && in_the_loop() && is_main_query() ) {
    // Add query arguments: action, post.
    $url = add_query_arg(
      [
        'action' => 'wporg_frontend_delete',
        'post'   => get_the_ID(),
      ], home_url()
    );

    return $content . ' <a href="' . esc_url( $url ) . '">' . esc_html__( 'Delete Post', 'wporg' ) . '</a>';
  }

  return null;
}

/**
 * Request handler
 */
function wporg_delete_post() {
  if ( isset( $_GET['action'] ) && 'wporg_frontend_delete' === $_GET['action'] ) {

    // Verify we have a post id.
    $post_id = ( isset( $_GET['post'] ) ) ? ( $_GET['post'] ) : ( null );

    // Verify there is a post with such a number.
    $post = get_post( (int) $post_id );
    if ( empty( $post ) ) {
      return;
    }

    // Delete the post.
    wp_trash_post( $post_id );

    // Redirect to admin page.
    $redirect = admin_url( 'edit.php' );
    wp_safe_redirect( $redirect );

    // We are done.
    die;
  }
}

/**
 * Add the delete link to the end of the post content.
 */
add_filter( 'the_content', 'wporg_generate_delete_link' );

/**
 * Register our request handler with the init hook.
 */
add_action( 'init', 'wporg_delete_post' );
```

### Restricted to a Specific Capability

The example above allows any visitor to the site to click on the "Delete" link and trash the post. However, we only want Editors and above to be able to click on the "Delete" link.

To accomplish this, we will check that the current user has the capability `edit_others_posts`, which only Editors or above would have:

```
/**
 * Generate a Delete link based on the homepage url.
 *
 * @param string $content   Existing content.
 *
 * @return string|null
 */
function wporg_generate_delete_link( $content ) {
  // Run only for single post page.
  if ( is_single() && in_the_loop() && is_main_query() ) {
    // Add query arguments: action, post.
    $url = add_query_arg(
      [
        'action' => 'wporg_frontend_delete',
        'post'   => get_the_ID(),
      ], home_url()
    );

    return $content . ' <a href="' . esc_url( $url ) . '">' . esc_html__( 'Delete Post', 'wporg' ) . '</a>';
  }

  return null;
}

/**
 * Request handler
 */
function wporg_delete_post() {
  if ( isset( $_GET['action'] ) && 'wporg_frontend_delete' === $_GET['action'] ) {

    // Verify we have a post id.
    $post_id = ( isset( $_GET['post'] ) ) ? ( $_GET['post'] ) : ( null );

    // Verify there is a post with such a number.
    $post = get_post( (int) $post_id );
    if ( empty( $post ) ) {
      return;
    }

    // Delete the post.
    wp_trash_post( $post_id );

    // Redirect to admin page.
    $redirect = admin_url( 'edit.php' );
    wp_safe_redirect( $redirect );

    // We are done.
    die;
  }
}

/**
 * Add delete post ability
 */
add_action('plugins_loaded', 'wporg_add_delete_post_ability');

function wporg_add_delete_post_ability() {    
  if ( current_user_can( 'edit_others_posts' ) ) {
    /**
     * Add the delete link to the end of the post content.
     */
    add_filter( 'the_content', 'wporg_generate_delete_link' );

    /**
     * Register our request handler with the init hook.
     */
    add_action( 'init', 'wporg_delete_post' );
  }
}
```