---
title: "术语拆分 (WordPress 4.2)"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Split Terms Wp 4 2
    - Taxonomies
    - Repos
---

# 术语拆分 (WordPress 4.2)

此处信息仅为历史记录。如果您对 2015 年之前术语的工作方式不感兴趣，可以跳过本节。

## WordPress 4.2 之前

不同分类法中具有相同缩略名的术语共享同一个术语 ID。例如，缩略名为 "news" 的标签和类别拥有相同的术语 ID。

## WordPress 4.2+

从 4.2 版本开始，当这些共享分类项中的一个被更新时，它将被拆分：更新后的分类项将被分配一个新的分类项 ID。

## 这对您意味着什么？

在绝大多数情况下，此更新是无缝且顺利的。然而，一些在选项、文章元数据、用户元数据或其他地方存储分类项 ID 的插件和主题可能受到了影响。

## 处理拆分

WordPress 4.2 包含两种不同的工具，以帮助插件和主题作者进行过渡。

### `split_shared_term` 钩子

当一个共享术语被分配新的术语 ID 时，会触发一个新的 `split_shared_term` 动作。

以下是插件和主题作者如何利用此钩子来确保存储的术语 ID 得到更新的几个示例。

#### Term ID stored in an option

Let's say your plugin stores an option called `featured_tags` that contains an array of term IDs (`[4, 6, 10]`) that serve as the query parameter for your homepage featured posts section.

In this example, you'll hook to `split_shared_term` action, check whether the updated term ID is in the array, and update if necessary.

```
/**
 * Update featured_tags option when a shared term gets split.
 *
 * @param int    $term_id          ID of the formerly shared term.
 * @param int    $new_term_id      ID of the new term created for the $term_taxonomy_id.
 * @param int    $term_taxonomy_id ID for the term_taxonomy row affected by the split.
 * @param string $taxonomy         Taxonomy for the split term.
 */
function wporg_featured_tags_split( int $term_id, int $new_term_id, int $term_taxonomy_id, string $taxonomy ): void {
  // we only care about tags, so we'll first verify that the taxonomy is post_tag.
  if ( 'post_tag' === $taxonomy ) {

    // get the currently featured tags.
    $featured_tags = get_option( 'featured_tags' );

    // if the updated term is in the array, note the array key.
    $found_term = array_search( $term_id, $featured_tags, true );
    if ( false !== $found_term ) {

      // the updated term is a featured tag! replace it in the array, save the new array.
      $featured_tags[ $found_term ] = $new_term_id;
      update_option( 'featured_tags', $featured_tags );
    }
  }
}
add_action( 'split_shared_term', 'wporg_featured_tags_split', 10, 4 );
```

#### Term ID stored in post meta

Let's say your plugin stores a term ID in post meta for pages so that you can show related posts for a certain page.

In this case, you need to use the `get_posts()` function to get the pages with your `meta_key` and update the `meta_value` matching the split term ID.

```
/**
 * Update related posts term ID for pages
 *
 * @param int    $term_id          ID of the formerly shared term.
 * @param int    $new_term_id      ID of the new term created for the $term_taxonomy_id.
 * @param int    $term_taxonomy_id ID for the term_taxonomy row affected by the split.
 * @param string $taxonomy         Taxonomy for the split term.
 */
function wporg_page_related_posts_split( int $term_id, int $new_term_id, int $term_taxonomy_id, string $taxonomy ): void {
  // find all the pages where meta_value matches the old term ID.
  $page_ids = get_posts(
    array(
      'post_type'  => 'page',
      'fields'     => 'ids',
      'meta_key'   => 'meta_key',
      'meta_value' => $term_id,
    )
  );

  // if such pages exist, update the term ID for each page.
  if ( $page_ids ) {
    foreach ( $page_ids as $id ) {
      update_post_meta( $id, 'meta_key', $new_term_id, $term_id );
    }
  }
}
add_action( 'split_shared_term', 'wporg_page_related_posts_split', 10, 4 );
```

### The `wp_get_split_term` function

[info]Using the `split_shared_term` hook is the preferred method for processing Term ID changes.

However, there may be cases where Terms are split without your plugin having a chance to hook to the `split_shared_term` action.[/info]

WordPress 4.2 stores information about taxonomy terms that have been split, and provides the `wp_get_split_term()` utility function to help developers retrieve this information.

Consider the case above, where your plugin stores term IDs in an option named `featured_tags`. You may want to build a function that validates these tag IDs (perhaps to be run on plugin update), to be sure that none of the featured tags has been split:

```
/**
 * Retrieve information about split terms and udpates the featured_tags option with the new term IDs.
 *
 * @return void
 */
function wporg_featured_tags_check_split() {
  $featured_tag_ids = get_option( 'featured_tags', array() );

  // check to see whether any IDs correspond to post_tag terms that have been split.
  foreach ( $featured_tag_ids as $index => $featured_tag_id ) {
    $new_term_id = wp_get_split_term( $featured_tag_id, 'post_tag' );

    if ( $new_term_id ) {
      $featured_tag_ids[ $index ] = $new_term_id;
    }
  }

  // save
  update_option( 'featured_tags', $featured_tag_ids );
}
```

Note that `wp_get_split_term()` takes two parameters, `$old_term_id` and `$taxonomy` and returns an integer.

If you need to retrieve a list of all split terms associated with an old Term ID, regardless of taxonomy, use `wp_get_split_terms()`.