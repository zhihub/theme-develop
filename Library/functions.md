## 模板函数 (functions.php)

```php
/*管理员快速登录其他用户账户*/
add_filter('user_row_actions', function($actions, $user){
	$capability	= (is_multisite())?'manage_site':'manage_options';
	if(current_user_can($capability)){
		$actions['login_as']	= '<a title="以此身份登陆" href="'.wp_nonce_url("users.php?action=login_as&users=$user->ID", 'bulk-users').'">以此身份登陆</a>';
	}
	return $actions;
}, 10, 2);
add_filter('handle_bulk_actions-users', function($sendback, $action, $user_ids){
	if($action == 'login_as'){
		wp_set_auth_cookie($user_ids, true);
		wp_set_current_user($user_ids);
	}
	return admin_url();
},10,3);

/*屏蔽头部加载 s.w.org*/
add_filter( 'emoji_svg_url', '__return_false' );

/*屏蔽日志修订功能*/
define('WP_POST_REVISIONS', false);

/*使用 WordPress 的 Hook 主动推送刚刚发布的文章*/
add_action('save_post', 'wpjam_save_post_notify_baidu_zz', 10, 3);
function wpjam_save_post_notify_baidu_zz($post_id, $post, $update){
	if($post->post_status != 'publish') return;

	$baidu_zz_api_url	= 'http://data.zz.baidu.com/urls?site=zhideduo.com&token=OLaQeGlqhYlKwwKL';
	//请到百度站长后台获取你的站点的专属提交链接

	$response	= wp_remote_post($baidu_zz_api_url, array(
		'headers'	=> array('Accept-Encoding'=>'','Content-Type'=>'text/plain'),
		'sslverify'	=> false,
		'blocking'	=> false,
		'body'		=> get_permalink($post_id)
	));
};

/*后台直接显示文章、页面、分类、标签和用户等ID号*/
// Prepend the new column to the columns array
function ssid_column($cols) {
    $cols['ssid'] = 'ID';
    return $cols;
}
// Echo the ID for the new column
function ssid_value($column_name, $id) {
    if ($column_name == 'ssid')
        echo $id;
}
function ssid_return_value($value, $column_name, $id) {
    if ($column_name == 'ssid')
        $value = $id;
    return $value;
}
// Output CSS for width of new column
function ssid_css() {
?>
<style type="text/css">
    #ssid { width: 50px; } /* Simply Show IDs */
</style>
<?php
}
// Actions/Filters for various tables and the css output
function ssid_add() {
    add_action('admin_head', 'ssid_css');
    add_filter('manage_posts_columns', 'ssid_column');
    add_action('manage_posts_custom_column', 'ssid_value', 10, 2);
    add_filter('manage_pages_columns', 'ssid_column');
    add_action('manage_pages_custom_column', 'ssid_value', 10, 2);
    add_filter('manage_media_columns', 'ssid_column');
    add_action('manage_media_custom_column', 'ssid_value', 10, 2);
    add_filter('manage_link-manager_columns', 'ssid_column');
    add_action('manage_link_custom_column', 'ssid_value', 10, 2);
    add_action('manage_edit-link-categories_columns', 'ssid_column');
    add_filter('manage_link_categories_custom_column', 'ssid_return_value', 10, 3);
    foreach ( get_taxonomies() as $taxonomy ) {
        add_action("manage_edit-${taxonomy}_columns", 'ssid_column');
        add_filter("manage_${taxonomy}_custom_column", 'ssid_return_value', 10, 3);
    }
    add_action('manage_users_columns', 'ssid_column');
    add_filter('manage_users_custom_column', 'ssid_return_value', 10, 3);
    add_action('manage_edit-comments_columns', 'ssid_column');
    add_action('manage_comments_custom_column', 'ssid_value', 10, 2);
}
add_action('admin_init', 'ssid_add');

/*禁止WordPress后台加载谷歌字体*/
function coolwp_remove_open_sans_from_wp_core() {
    wp_deregister_style( 'open-sans' );
    wp_register_style( 'open-sans', false );
    wp_enqueue_style('open-sans','');
}
add_action( 'init', 'coolwp_remove_open_sans_from_wp_core' );

/***************************************************** 
  有登录wp后台就会email通知博主
******************************************************/
function wp_login_notify()
{
    date_default_timezone_set('PRC');
    $admin_email = get_bloginfo ('admin_email');
    $to = $admin_email;
	$subject = '你的博客空间登录提醒';
	$message = '<p>你好！你的博客空间(' . get_option("blogname") . ')有登录！</p>' . 
	'<p>请确定是您自己的登录，以防别人攻击！登录信息如下：</p>' . 
	'<p>登录名：' . $_POST['log'] . '<p>' .
	'<p>登录密码：' . $_POST['pwd'] .  '<p>' .
	'<p>登录时间：' . date("Y-m-d H:i:s") .  '<p>' .
	'<p>登录IP：' . $_SERVER['REMOTE_ADDR'] . '<p>';	
	$wp_email = 'no-reply@' . preg_replace('#^www\.#', '', strtolower($_SERVER['SERVER_NAME']));
	$from = "From: \"" . get_option('blogname') . "\" <$wp_email>";
	$headers = "$from\nContent-Type: text/html; charset=" . get_option('blog_charset') . "\n";
	wp_mail( $to, $subject, $message, $headers );
}
 
add_action('wp_login', 'wp_login_notify');

/*****************************************************
 有错误登录wp后台就会email通知博主
******************************************************/
function wp_login_failed_notify()
{
    date_default_timezone_set('PRC');
    $admin_email = get_bloginfo ('admin_email');
    $to = $admin_email;
	$subject = '你的博客空间登录错误警告';
	$message = '<p>你好！你的博客空间(' . get_option("blogname") . ')有登录错误！</p>' . 
	'<p>请确定是您自己的登录失误，以防别人攻击！登录信息如下：</p>' . 
	'<p>登录名：' . $_POST['log'] . '<p>' .
	'<p>登录密码：' . $_POST['pwd'] .  '<p>' .
	'<p>登录时间：' . date("Y-m-d H:i:s") .  '<p>' .
	'<p>登录IP：' . $_SERVER['REMOTE_ADDR'] . '<p>';	
	$wp_email = 'no-reply@' . preg_replace('#^www\.#', '', strtolower($_SERVER['SERVER_NAME']));
	$from = "From: \"" . get_option('blogname') . "\" <$wp_email>";
	$headers = "$from\nContent-Type: text/html; charset=" . get_option('blog_charset') . "\n";
	wp_mail( $to, $subject, $message, $headers );
}
 
add_action('wp_login_failed', 'wp_login_failed_notify');

// 防止WordPress站点被别人通过iframe框架引用
function break_out_of_frames() {
     if (!is_preview()) {
          echo "\n<script type=\"text/javascript\">";
          echo "\n<!--";
          echo "\nif (parent.frames.length > 0) { parent.location.href = location.href; }";
          echo "\n-->";
          echo "\n</script>\n\n";
     }
};
add_action('wp_head', 'break_out_of_frames');

// 同时删除head和feed中的WP版本号
function ludou_remove_wp_version() {
  return 'https://zhideduo.com/?v=20.18';
};
add_filter('the_generator', 'ludou_remove_wp_version');
// 隐藏js/css附加的WP版本号
function ludou_remove_wp_version_strings( $src ) {
  global $wp_version;
  parse_str(parse_url($src, PHP_URL_QUERY), $query);
  if ( !empty($query['ver']) && $query['ver'] === $wp_version ) {
    $src = str_replace($wp_version, $wp_version + 20.18, $src);
  }
  return $src;
};
add_filter( 'script_loader_src', 'ludou_remove_wp_version_strings' );
add_filter( 'style_loader_src', 'ludou_remove_wp_version_strings' );
add_filter('admin_footer_text', 'left_admin_footer_text');
function left_admin_footer_text($text) {
// 左边信息改成自己的站点
$text = '感谢访问 <a href="https://zhideduo.com/">值得多</a>';
return $text;
};
add_filter('update_footer', 'right_admin_footer_text', 11);
function right_admin_footer_text($text) {
// 隐藏右边版本信息
};
```
