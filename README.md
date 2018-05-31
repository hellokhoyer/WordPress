# Theme Development Handbook

----------


### Theme bootstrapping: ###
আমাদের থিমের বুটস্ট্রাপিং করতে প্রথমেই functions.php তে কিছু ফাংশনালিটি অ্যাড করতে হয়। এইগুলোকে বুটস্ট্রাপিং অর্থাৎ থিম তৈরির প্রথম স্টেপ বলে থাকি।

    <?php
	function theme_bootstrapping(){
		load_theme_textdomain("domain_name"); //ready to translate theme.
		add_theme_support("post-thumbnails"); //for adding featured image
		add_theme_support("title-tag");
		register_nav_menu("topmenu", __("Top Menu", "text-domain_name")); //topmenu is location name.
		register_nav_menu("footermenu", __("Top Menu", "text-domain_name")); //topmenu is location name.
	}
    add_action('after_setup_theme', 'theme_bootstrapping');

> এখানে থিমের ডোমেইন নেম ফাংশন prefix হিসেবে থাকবে।



### Style and Script Enqueue: ###

wordpress.org এবং themeforest এর নিয়ম অনুযায়ী থিম এর সকল including ফাইল আলাদা ফোল্ডার এ রাখতে হয়। এ জন্য আমাদের একটা ফোল্ডার তৈরি করতে হবে। তো এখন আমরা inc নামের একটা ফোল্ডারে enqueue.php তে সকল ফাইল রাখবো। তো প্রথমেই আমাদের enqueue.php ফাইলটি থিমের functions.php ফাইলের সাথে যুক্ত করতে হবে। 

    require_once(get_template_directory() . 'inc/enqueue.php');

এবার enqueue.php তে আমাদের style এবং script ফাইল লিঙ্ক করবো।

     function theme_assets() {
     wp_enqueue_style( 'custom-style', get_stylesheet_uri() );
     wp_enqueue_style( "owltheme", get_template_directory_uri()."/owl.theme.default.min.css", null );
     wp_enqueue_script( 'script-name', get_template_directory_uri().'/js/example.js', array(), '1.0.0', true );
	}

	add_action('wp_enqueue_scripts', 'theme_assets');

> এখানে থিমের ডোমেইন নেম ফাংশন prefix হিসেবে থাকবে।
> 
> WordPress 4.7 থেকে `get_theme_file_url("file-path")` ব্যবহার করা হয়।


----------

#### Cache Busting: ####


ডেভেলপমেন্ট এর সময় আমাদের ভার্সন কন্ট্রোল করা বড় একটা সমস্যা। অনেকসময় আমাদের এডিটগুলো সাথে সাথে দেখতে পারিনা। ক্যাশ নিয়ে নেয়। যার জন্য ক্যাশ বাস্টিং করাটা অনেক জরুরী। এজন্য একটা চমতকার ট্যাকনিক ফলো করতে পারি। PHP এর `"unix epoch"` সিস্টেম। যেটা আমরা `time()` ফাংশন হিসেবে চিনি। ভার্সন এর জায়গায় এটা বসিয়ে দিলেই হবে।

    wp_enqueue_style( 'stylesheet', get_stylesheet_uri(), null, titme());

তবে এটার একটা অসুবিধা আছে। আমরা ডেভেলপমেন্ট ডোমেইন থেকে যখন সার্ভারে আপলোড দিবো। ইউজাররা ভালভাবে থিমের কারেন্ট ভার্সন দেখতে পারবে না। এর জন্য আমাদের ডেভেলপমেন্ট এর জন্য শধু এই সিস্টেমটা রাখবো। তো এর একটা ভালো সমাধান আছে। যার মাধ্যমে আমরা `style.css` থেকে থিমের ভার্সন নিয়ে আসতে পারবো।

    if(site_url()=="localhost/theme_path"){
		define("VERSION", time());
	} else {
		define("VERSION", wp_get_theme()->get("version"));
	}

> এখানে থিমের লোকাল পাথ দিয়ে লোপ চালানো হয়েছে।
> 
> `wp_get_theme()` ফাংশন দিয়ে থিম ফাইলে access করে। অবজেক্ট তৈরি করে থিমের স্টাইলশিটের `Version` কল করেছি।


এবার থিমের মধ্যে ভার্সন এর জায়গায় `VERSION` ভ্যারিয়েবলটা ডিফাইন করবোঃ
    
    wp_enqueue_style( 'stylesheet', get_stylesheet_uri(), null, VERSION);

----------


### Menu Register: ###
নেভিগেশন মেনু তৈরি করতে আমাদের থিম অপশন থেকে প্রথমে মেনু সাপোর্ট দিতে হবে। থিম bootstrapping এ এই ফাংশন কল করতে হবে।

     	register_nav_menu("topmenu", __("Top Menu", "text-domain_name")); //topmenu is location name.
		register_nav_menu("footermenu", __("Top Menu", "text-domain_name")); //bottommenu is location name.

এখন আমরা সাইটের ফ্রন্টেন্ডে মেনু নিয়ে আসবো। header.php তে কিছু কোড কোড অ্যাড করবো।

    <?php
	wp_nav_menu(
		array(
				'theme_location' => 'footermenu', //location from bootstrapping
				'menu_class' => '' //ul class goes here.
			 )
	);
	?>



----------


### Create Widget: ###

প্রথমেই থিমের functions.php তে সাইডবার রেজিস্টার করে দিতে হবে।

	function theme_sidebar {
    	register_sidebar(
			array(
					'name'          => esc_html__( 'Sidebar', 'theme-prefix' ),
					'id'            => 'sidebar_name',
					'description'   => esc_html__( 'Add widgets here.', 'theme-prefix' ),
					'before_widget' => '<div id="%1$s" class="widget %2$s">',
					'after_widget'  => '</div>',
					'before_title'  => '<h2 class="widget-title">',
					'after_title'   => '</h2>',
				)
		);
	}

	add_action('widget_init', 'theme_sidebar');

> before এবং after এ আপনার ইচ্ছামত ফ্রন্টেন্ড ক্লাস অ্যাড করে দিতে পারবেন।

> এখানে থিমের ডোমেইন নেম ফাংশন prefix হিসেবে থাকবে।

এবার সাইটের ফ্রন্টেন্ডে সাইডবার কল করবো। তো সাইটে সাইডবার কল করলে অবশ্যই তা কন্ডিশনাল মেথড ইউজ করে দিতে হবে।

    		<?php if(is_active_sidebar('sidebar_name')) : ?>
	            <div>
	                <?php dynamic_sidebar('sidebar_name'); ?>
	            </div>
            <?php endif; ?>


----------



### Create Shortcode: ###

থিম ফরেস্টের রেস্ট্রিকশন অনুযায়ী থিমের সাথে কোনো শর্টকোড রাখা যাবে না। তো এ জন্য আমাদের শর্টকোড কে আলাদা প্লাগিন হিসেবে ইউজ করতে হবে। প্লাগিন একটিক করে শর্টকোড ইউজ করবো।

    
	function alert_shortcode($atts, $content = null)
    {
        extract(shortcode_atts(array(
        'type' => 'alert-success',
        'text' => ''
        ), $atts));

        return '<div class="alert '.esc_attr($type).'" role="alert">'.esc_html($content).'</div>';
    }
	add_shortcode('alert', 'alert_shortcode');
> শর্টকোড লিখার আগে আমাদের থিম নেম frefix হিসেবে ইউজ করতে হবে।

##### একটু এডভান্সড লেভেল এর শর্টকোডের মাধ্যমে ইউজ করতে হলে #####

    function post_list_shortcode($atts){
    extract( shortcode_atts( array(
        'count' => -1,
        'type' => 'post'
    ), $atts) );
     
    $q = new WP_Query(
        array(
            'posts_per_page' => -1,
            'post_type' => $type
            )
        );      
         
    	$list = '<ul>';
    while($q->have_posts()) : $q->the_post();
        $idd = get_the_ID();
        $post_content = get_the_content();
        $list .= '<li><a href="'.get_permalink().'">'.get_the_title().'</a></li>';        
    endwhile;
    	$list.= '</ul>';
    	wp_reset_query();
    	return $list;
	}
	add_shortcode('post_list', 'post_list_shortcode');

##### Visual Composer Addon with Shortcode #####


	function vc_post_list_addon(){

	vc_map( array(
        "name" => __( "post list", "my-text-domain" ),
        "base" => "post_list",
        "params" => array(
           array(
              "type" => "textfield",
              "heading" => __( "post count", "my-text-domain" ),
              "param_name" => "count",
              "value" => __( "Default param value", "my-text-domain" ),
              "description" => __( "Description for foo param.", "my-text-domain" )
           ),
           array(
            "type" => "textfield",
            "heading" => __( "post count", "my-text-domain" ),
            "param_name" => "count",
            "value" => __( -1, "my-text-domain" ),
            "description" => __( "this is description", "my-text-domain" )
         )
        )
	));
    
	}
	add_action('vc_before_init', 'vc_post_list_addon');

----------


#### More Advanced Shortcode And Plugin Snipet ####

    <?php

	 /*

	Plugin Name: Theme Toolkit

	 */

	//exit if acceced directly
	if ( ! defined('ABSPATH') ) {
    exit;
	}

	//define
	define ('THEME_ACC_URL', WP_PlUGIN_URL . '/' . plugin_basename( dirname	(__FILE__) ) . '/');
	define ('THEME_ACC_PATH', plugin_dir_path( __FILE__ ));

	function my_theme_custom_post() {
    register_post_type( 'testimonial',
        array(
            'labels' => array(
                name => __('testimonials'),
                singular_name => __('testimonial')
            ),
            'supports' => array('title', 'editor', 'thumbnail', 'page-attributes'),
            'public' => false,
            'show_ui' => true,

        ) 
    );
	}

	//print shortcode in widget
	add_filter('widget_text', 'do_shortcode');

	//loading vc addons
	require_once( THEME_ACC_PATH . 'vc-addons/vc-blocks-load.php' );

	//theme_shortcode
	require_once( THEME_ACC_PATH . 'theme-shortcodes/slides-shortcode.php' );

	//shortcode depended on visual composer
	include_once( ABSPATH . 'wp-admin/includes/plugin.php' );
	if (is_plugin_active('js_composer/js_composer.php')){
    require_once( THEME_ACC_PATH . 'theme_shortcodes/staff-shortcode.php' );
	}

	//registering theme toolkit file
	function theme_toolkit_files(){
    wp_enqueue_style( 'owl_carousel', plugin_dir_url(__FILE__) . 'assets/css/owl-carousel.css' );
    wp_enqueue_script( 'owl_carousel', plugin_dir_url(__FILE__) . 'assets/js/owl-carousel.js', array('jquery'), '', true);
	}
	add_action('wp_enqueue_scripts', 'theme_toolkit_files');

	?>