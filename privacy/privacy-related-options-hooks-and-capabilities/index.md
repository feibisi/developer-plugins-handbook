---
title: "隐私相关选项、钩子和能力"
post_status: publish
comment_status: open
taxonomy:
  category:
    - developer-plugins-handbook
  post_tag:
    - Privacy Related Options Hooks And Capabilities
    - Privacy
    - Repos
---

# 隐私相关选项、钩子和能力

隐私工具最初在 WordPress 4.9.6 版本中引入。这些工具旨在允许（并鼓励）开发者将其用作隐私导出器、隐私擦除器和隐私政策指南的一部分。

此后，引入了若干新钩子以扩展可用功能。这些钩子允许开发者在导出和擦除请求中包含额外的个人数据，并为隐私政策指南提供建议内容。

除了控制这些工具的能力外，还新增了多个用于请求和确认邮件的过滤器，从而实现对通知的更精细控制。

## 选项

- `wp_page_for_privacy_policy` – 包含网站隐私政策页面的页面 ID。

## 操作

- `user_request_action_confirmed` – 当用户确认隐私请求时触发。
- `wp_privacy_delete_old_export_files` – 一个计划任务，用于从个人数据导出文件夹中清理旧的导出文件。
- `wp_privacy_personal_data_erased` – 在最后一个擦除器的最后一页完成后触发。
- `wp_privacy_personal_data_export_file` – 用于在导出流程中创建个人数据导出文件。
- `wp_privacy_personal_data_export_file_created` – 在个人数据导出文件创建后触发。

## 过滤器

- `privacy_policy_url` – 过滤隐私政策页面的 URL。
- `the_privacy_policy_link` – 过滤隐私政策页面链接的 HTML。
- `wp_get_default_privacy_policy_content` – 过滤通过隐私政策指南建议包含的默认内容。
- `user_request_action_confirmed_message` – 允许修改向用户显示的操作确认消息。
- `user_request_action_description` – 过滤用户操作描述。
- `user_request_action_email_content` – 过滤尝试账户操作时发送的电子邮件正文。
- `user_request_action_email_headers` – 过滤尝试账户操作时发送的电子邮件标头。
- `user_request_action_email_subject` – 过滤尝试账户操作时发送的电子邮件主题。
- `user_request_confirmed_email_content` – 过滤用户请求确认电子邮件的正文。
- `user_request_confirmed_email_headers` – 过滤用户请求确认电子邮件的标头。
- `user_request_confirmed_email_subject` – 过滤用户请求确认电子邮件的主题。
- `user_request_confirmed_email_to` – 过滤数据请求确认通知的收件人。
- `user_request_key_expiration` – 过滤用户请求确认密钥的过期时间。
- `wp_privacy_additional_user_profile_data` – 用于扩展隐私导出器的用户个人资料数据的过滤器。
- `wp_privacy_export_expiration` – 控制导出文件允许保留的时间，默认为 3 天。
- `wp_privacy_personal_data_email_content` – 允许修改发送给用户的包含其个人数据导出文件链接的电子邮件消息。
- `wp_privacy_personal_data_email_headers` – 过滤随个人数据导出文件发送的电子邮件的标头。
- `wp_privacy_personal_data_email_subject` – 过滤导出请求完成时发送的电子邮件的主题。
- `wp_privacy_personal_data_email_to` – 过滤个人数据导出电子邮件通知的收件人。

[info]使用 `wp_privacy_personal_data_email_to` 时应格外谨慎，以避免将数据导出链接发送到错误的收件人电子邮件地址。[/info]

- `wp_privacy_personal_data_erasers` – 支持注册核心和插件个人数据擦除器。
- `wp_privacy_personal_data_erasure_page` – 过滤个人数据擦除器数据页面。允许除 AJAX 外，其他目的地也能使用擦除响应。
- `wp_privacy_personal_data_exporters` – 支持注册核心和插件个人数据导出器。
- `wp_privacy_personal_data_export_page` – 过滤个人数据导出器数据页面。用于构建导出报告。允许除 AJAX 外，其他目的地也能使用导出响应。
- `wp_privacy_anonymize_data` – 过滤每种类型的匿名数据。
- `wp_privacy_exports_dir` – 过滤用于存储个人数据导出文件的目录。
- `wp_privacy_exports_url` – 过滤用于存储个人数据导出文件的目录 URL。
- `user_confirmed_action_email_content` – 过滤用户请求确认邮件的正文。当用户请求被确认时，该邮件会发送给管理员。
- `user_erasure_fulfillment_email_to` – 过滤数据擦除完成通知的收件人。
- `user_erasure_complete_email_subject` – 过滤擦除请求完成时发送的邮件主题。
- `user_confirmed_action_email_content` – 过滤数据擦除完成通知的正文。当管理员完成用户的数据擦除请求时，该邮件会发送给用户。
- `user_erasure_complete_email_headers` – 过滤数据擦除完成通知的邮件头。

## 功能权限

隐私工具的访问权限由几个新增功能控制。管理员（在非多站点安装中）默认拥有这些权限。具体权限如下：

- `erase_others_personal_data` – 控制“工具”菜单下是否显示“擦除个人数据”子菜单
- `export_others_personal_data` – 控制“工具”菜单下是否显示“导出个人数据”子菜单  
- `manage_privacy_options` – 控制“设置”菜单下是否显示“隐私”子菜单