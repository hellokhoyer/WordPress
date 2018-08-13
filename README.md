# Theme Development Handbook

----------

----------


## How can i help you? ##
- **[Theme bootstrapping](#theme-bootstrapping)**
- **[Style and Script Enqueue](#style-and-script-enqueue)**
- **[Cache Busting](#cache-busting)**
- **[Menu Register](#menu-register)**
- **[Create Widget](#create-widget)**
- **[Custom Post Type](#custom-post-type)**
- **[Codestar Framework](#codestar-framework)**
- **[CMB2](#CMB2)**



----------


### Theme bootstrapping: ###
আমাদের থিমের বুটস্ট্রাপিং করতে প্রথমেই functions.php তে কিছু ফাংশনালিটি অ্যাড করতে হয়। এইগুলোকে বুটস্ট্রাপিং অর্থাৎ থিম তৈরির প্রথম স্টেপ বলে থাকি।

    <?php
	function theme_bootstrapping(){
		load_theme_textdomain("domain_name"); //ready to translate theme.
		add_theme_support("post-thumbnails"); //for adding featured image
		add_theme_support("title-tag");
		register_nav_menu("topmenu", __("Top Menu", "text-domain_name")); //topmenu is location name.
		register_nav_menu("footermenu", __("Footer Menu", "text-domain_name")); //footermenu is location name.
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

### Cache Busting: ###


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
		register_nav_menu("footermenu", __("Bottom Menu", "text-domain_name")); //bottommenu is location name.

এছাড়াও আপনি চাইলে array ব্যবহার করে একাধিক মেনু অ্যাড করতে পারবেন।

    register_nav_menus( array(
    'top_menu' => __( 'Top Menu', 'text_domain' ),
    'footer_menu' => __( 'Footer Menu', 'text_domain' )
    ) );
    

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

### Custom Post Type: ###

কাষ্টম পোষ্ট টাইপ আমাদের অনেক ক্ষেত্রেই দরকার হয়। খুব সহজ কয়েকটি স্টেপ ফলো করে আমরা কাষ্টম পোষ্ট টাইপ রেজিস্টার করতে পারি।

প্রথমেই আমাদের `functions.php` তে `WordPress` এর একটা বিল্টইন ফাংশন কল করবো।

    register_post_type('', '');

> এখানে দুইটা আর্গুমেন্ট। ১ম টা হচ্ছে পোষ্টটাইপ নেম। আর ২য় টা তে প্রয়োজনীয় প্রোপার্টি।

এখন নাম এবং প্রোপার্টি গুলো অ্যাড করবো। যেহেতু অনেকগূলো প্রোপার্টি তাই আমরা অ্যারে আকারে লিখবো।

    register_post_type('name', array(

	));

এখন রিলোড দিলে দেখতে পাবেন আমাদের পোষ্ট টাইপ টি ড্যাশবোর্ড এ আসে নি। এ জন্য `'public' => true` করে দিতে হবে। যাতে ইউজার এক্সেস করতে পারে।

    
    register_post_type('Name', array(
		'public' => true,
	));

এখন আমাদের পোষ্ট টাইপ চলে আসলো। কিন্ত লেভেল গূলো চেঞ্জ হয় নি। এখন আমরা লেভেল চেঞ্জ করবো।

    register_post_type('name', array(
		'public' => true,
		'labels' => array(
					'name' => "Name",
					'all_items' => "All Names",
					'add_new' => 'Add New Name'
		)
	));

এখন পোষ্ট টাইপ তৈরি শেষ। আমরা [Dashicon](https://developer.wordpress.org/resource/dashicons/ "dashicon") থেকে একটা আইকন দিয়ে দিবো। এবং এটা অবশ্যই লেভেল এর বাইরে।


    register_post_type('name', array(
		'public' => true,
		'labels' => array(
					'name' => "Name",
					'all_items' => "All Names",
					'add_new' => 'Add New Name'
		),

		'menu_icon' => 'icon url'
	));
> name এর জায়গায় আমাদের পোষ্ট টাইপ নেম। singular plural এর ব্যপারগুলো খেয়াল করবেন।



আপাতত কাজ শেষ। এখন এডিটরে যাবো। আমরা `supports` এর মাধ্যমে আমাদের প্রয়োজনীয় যা যা লাগে অ্যাড করবো।

    register_post_type('name', array(
		'public' => true,
		'labels' => array(
					'name' => "Name",
					'all_items' => "All Names",
					'add_new' => 'Add New Name'
		),

		'menu_icon' => 'icon url',
		'supports' => array('', '', '')
	));

> এখানে ৩টা ভ্যালূ। ১ম টা হচ্ছে `title` ২য় টা `editor` এবং ৩য় টা `featured image`

যেহেতু আমরা কিছুই অ্যাড করি নি তো আমাদের পেইজটা পুরো ফাকা। তাই কিছু সাপোর্টস অ্যাড করবো। এবং পজিশন ডিক্লেয়ার করে দেবো। অর্থাৎ ড্যাসবোর্ড এর কোথায় আমাদের পোষ্ট টাইপটি বসবে।


    register_post_type('name', array(
		'public' => true,
		'labels' => array(
					'name' => "Name",
					'all_items' => "All Names",
					'add_new' => 'Add New Name'
		),

		'menu_icon' => 'icon url',
		'supports' => array('title', 'editor', 'thumbnail'),
		'menu_position' => 6
	));

এবার আমাদের ব্যাকএন্ড রেডি পোষ্ট পাবলিশ করার জন্য। কিছু পোষ্ট পাবলিশ করে নেই। এখন ব্যকএন্ডের কাজ প্রায় পুরোটাই শেষ। এখন আমরা ফ্রন্টেন্ডের কোন যায়গায়। এই কাষ্টম পোষ্টগুলো দেখাবো এটা নিয়ে কাজ করবো।

কাষ্টম পোষ্ট হোক আর যা ই হোক। আমাদের প্রথমে পোষ্টগুলো দেখাতে হবে। তো কাজটা করে ফেলি।

    <?php while(have_posts()) : the_post(); ?>
	//Our Design Content Here
	<?php endwhile; ?>
এখন আমরা টাইটেল পার্মালিঙ্ক ইমেজ সবগুলোর জন্য ফাংশন ইউজ করবো।

    <?php the_title() ?> //For title
    <?php the_permalink() ?> //For post url
    <?php the_post_thumbnail();  ?> //For image

এখন আমাদের শুধু রেগুলার পোষ্ট শো করানোর কাজটা শেষ হলো। কিন্ত এই পোষ্ট টাইপের পোষ্ট দেখাতে একটা অবজেক্ট তৈরি করতে হবে। যার মাধ্যমে ওয়ার্ডপ্রেস এর ক্লাস থেকে ডাটা কল করবো।

	$name = new WP_Query();
> new কিওয়ার্ড দিয়ে অবজেক্ট তৈরি করেছি। যেটা `$name` variable এ স্টোর করেছি। তো এখন থেকে `$name` অবজেক্ট হিসেবে কাজ করবে।

এখন কোন পোষ্ট টাইপের পোষ্ট দেখাবে। এটা তাকে বুঝিয়ে দিতে হবে। মাল্টিপল ভ্যালু হতে পারে। এজন্য একটা অ্যারে তৈরি করবো।


	$name = new WP_Query(array(
				'post_type' => 'name',
	));
> এখানে name হচ্ছে আমাদের রেজিষ্টার করা পোষ্ট টাইপ নেম।

এখন আমাদের সবগুলো পোষ্ট কন্টেন্টে অবজেক্ট অ্যাড করে দেই। method এর মাধ্যমে।

    <?php $name->the_title() ?> //For title
    <?php $name->the_permalink() ?> //For post url
    <?php $name->the_post_thumbnail();  ?> //For image



----------

### Codestar Framework ###
কোডস্টার ফ্রেমওয়ার্ক নিয়ে কাজ করতে গেলে বেশ কিছু স্টেপ ফলো করতে হয়। আমরা স্টেপ বাই স্টেপ সেগুলো দেখবো।

প্রথমেই আমরা ফ্রেমওয়ার্কটি ডাউনলোড করবো। তাদের [অফিশিয়াল গিট রিপোজিটরি](https://github.com/Codestar/codestar-framework/) থেকে।

দুই ভাবেই কাজ করা যায়। থিমের সাথে অথবা প্লাগিন হিসেবে। তো, এখন আমরা চাচ্ছি থিমের সাথে ইন্ট্রিগেট করে কাজ করতে।

প্রথমেই আমাদের ডাউনলোড করা zip ফাইলটিকে এক্সট্রাক্ট করবো। এবং আমাদের ওয়ার্ডপ্রেস থিম ডিরেক্টরিতে একটা ফোল্ডার ফোল্ডার ইনক্লুড করবো। এবং এক্সট্রাকটেড ফাইলগুলো ঐ ফোল্ডারে মুভ করবো।

আমাদের ফ্রেমওয়ার্কের প্রায় সকল ফাইল ইনক্লুডেড হয়ে গেছে। এখন মেইন যে ফাইল অর্থাৎ `cs-framework.php` কে `functions.php` তে লিঙ্ক করবো। যাতে আমাদের থিম ফাংশনের সাথে অ্যাড হয়ে যায়।

    require get_template_directory() . '/inc/cs-framework/cs-framework.php';

    
এখন রিলোড দিলে দেখতে পারবেন ফ্রেমওয়ার্কটি আমাদের থিমের সাথে অ্যাড হয়ে গেছে। আমাদের ইন্ট্রিগেশন পার্ট শেষ।

এখন আমরা নতুম একটা ফাইল ক্রিয়েট করে আমাদের নিজের মতো করে অপশন গুলো সাজাবো। এজন্য `metabox-options.php` নামে একটা ফাইল ক্রিয়েট করছি। অথবা যেকোনো নামে ক্রিয়েট করতে পারেন। কোনো সেকশন নিয়ে কাজ করতে সেই সেকশনের নামেও করতে পারেন।

তো আমরা যে নতুম ফাইলটি ক্রিয়েট করলাম এখন এটাকেও আমাদের `functions.php` তে কল করতে হবে।


    require get_template_directory() . '/inc/cs-framework/metabox-options.php';

এখন আমাদের নতুন ফাইলের ডিরেক্ট এক্সেস ব্লক করে দিতে হবে। এজন্য এ কোডগুলো লিখবো।

    <?php if ( ! defined('ABSPATH') ) { die; };

এখন একটা ফাংশন ক্রিয়েট করবো। যেখানে আমরা সমস্ত কাজ করবো।

    name_theme_metabox(){
    
    }
> name এর যায়গায় আপনার থিম নেম প্রিফিক্স হিসেবে হবে।

যেহেতু আমরা মেটাবক্স নিয়ে কাজ করবো। তো মেটাবক্স অপশন এর ফিল্টার হুকটি আমরা ইউজ করবো।

    name_theme_metabox(){
    
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');

এখন আমরা অপশনগুলোকে নিজের মতো কাস্টমাইজ করবো। এজন্য একটা আর্গুমেন্ট নেই।

    name_theme_metabox($options){
    
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');


এবার যেহেতু আমরা মেটাক্স তৈরি করবো সেজন্য আগের গুলো বা ডিফল্টগুলোকে রিসেট করতে হবে। এজন্য একটা ফাকা `array` দিবো।

    name_theme_metabox($options){
        $options = array();
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');

এখন যদি আমরা রিলোড করি দেখতে পাবো কোনো অপশনই নেই। এবার আমরা আমাদের মুল কাজগুলো করার জন্য রেডি। এবার `$options` আর্গুমেন্টটাকে এক্সট্রাক্ট করবো। নতুনভাবে ডাটা অ্যাড করবো। এজন্য একটা অ্যারে তৈরি করে নিতে হবে।

    name_theme_metabox($options){
        $options = array();

		$options[] = array();
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');

এবার আমরা ডাটা অ্যাড করবো। তার আগে অপশন এর একটা নাম দিয়ে দেই। এবং টাইটেল আইকন অ্যাড করে ফেলি।

    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name', //name should be unique.
			'title' => 'Our Section',
			'icon' => 'fa fa-icon'
		);
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');
> বামপাশের আইডিগুলো ইউনিক। যেটা কোডস্টারের প্যারামিটার।
> 
> section_name হচ্ছে আমাদের আইডি। অর্থাৎ যে নামে আমরা অপশনকে কল করবো মেটা ভ্যালূ বা ডাটা কালেক্ট করার জন্য। সেটা অবশ্যই ইউনিক রাখবো। 

আমাদের অপশন তৈরি শেষ। কিন্ত সেখানে কোনো ফিল্ড অ্যাড করা হয় নি। এবার কিছু ফিল্ড অ্যাড করবো। যেহেতু অনেকগুলো হবে। তাই অ্যারে আকারে লিখবো।


    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name', //name should be unique.
			'title' => 'Our Section',
			'icon' => 'fa fa-icon',
			'fields' => array(
				array(
				'id' => 'option_id',
				'type' => 'text',
				'title' => 'Add Text For First Option'
				),

				array(
				'id' => 'another_id',
				'type' => 'text',
				'title' => 'Add Text For Second Option'
				),
		);
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');

অপশন তৈরি হয়ে গেলো। এখন এই অপশনটা কোথায় দেখাবো তার জন্য পোষ্টটাইপ সিলেক্ট করে দিতে হবে।


    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name' //name should be unique.
			'title' => 'Our Section'
			'icon' => 'fa fa-icon'
			'post_type' => 'page',
			'fields' => array(
				array(
				'id' => 'option_id',
				'type' => 'text',
				'title' => 'Add Text For First Option'
				),

				array(
				'id' => 'another_id',
				'type' => 'text',
				'title' => 'Add Text For Second Option'
				),
			);
    	}
	add_filter('cs_metabox_options', 'name_theme_metabox');

এখন আমাদের কাজ শেষ। তবে তার আগে ফাংশনটাকে রিটার্ন করে নেবো।

    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name', //name should be unique.
			'title' => 'Our Section',
			'icon' => 'fa fa-icon',
			'post_type' => 'page',
			'fields' => array(
				array(
				'id' => 'option_id',
				'type' => 'text',
				'title' => 'Add Text For First Option'
				),

			array(
				'id' => 'another_id',
				'type' => 'text',
				'title' => 'Add Text For Second Option'
			),
		);
	return $options;
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');

আমাদের ব্যকএন্ড রেডি হয়ে গেলো কাজ করার জন্য।

এখন ফ্রন্টেন্ডে আমাদের অপশনগুলো অ্যাড করে দিবো। কোডস্টারের `cs_get_option('')` ফাংশনের মাধ্যমে।

    <h1><?php echo cs_get_option('field_id'); ?></h1>
> যে ডাটা ইম্পোর্ট করতে চাই। সেটা আইডিতে দিয়ে দিবো।



#### Types: ####

এখন আমাদের টাইপ সম্পর্কে জানাটা জরুরি। যেহেতু আমরা অ্যাডভান্সড কাজ করবো। আমি পর্যায়ক্রমে আমাদের প্রয়োজনীয় টাইপ দেখবো। যাতে প্রোজেক্ট করতে গিয়ে বাধাগ্রস্থ না হতে হয়।

একনজরে টাইপগুলো দেখে নেইঃ 

- text
- textarea
- select
	- options
- image


#### Create Group: ####

গ্রুপ তৈরি করতে হয় মুলতঃ ভিবিন্ন টিম মেম্বার সেকশন বা এই টাইপ কমন জিনিস নিয়ে যে ধরনের সেকশন তৈরি করা হয়। তো সবকিছুই আগের মতোই। শুধু ফিল্ড সেকশন এ গ্রুপ ডিফাইন করে দিতে হবে। তো আমরা আলাদা একটা অ্যারে নেবো।

    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name', //name should be unique.
			'title' => 'Our Section',
			'icon' => 'fa fa-icon',
			'fields' => array(
				array(
				'id' => 'our_field_group',
				'type' => 'group', //must be
				'title' => 'Add Text For Second Option'
				'button_title' => 'Add button', //must be
				'accordion_title' => 'Add new field' //must be
				),

		);
    }
	add_filter('cs_metabox_options', 'name_theme_metabox');

গ্রুপ তৈরি হলো কিন্ত ঐ গ্রুপে কোনো ফিল্ড নেই। এখন আমরা গ্রুপের জন্য ফিল্ড অ্যাড করবো।

    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name', //name should be unique.
			'title' => 'Our Section',
			'icon' => 'fa fa-icon',
			'fields' => array(
				array(
				'id' => 'our_field_group',
				'type' => 'group', //must be
				'title' => 'Add Text For Second Option'
				'button_title' => 'Add button', //must be
				'accordion_title' => 'Add new field' //must be
				),
				'fields' => array(
					array(
					'id' => 'option_id',
					'type' => 'text',
					'title' => 'Add Text For First Option'
					),

					array(
					'id' => 'another_id',
					'type' => 'text',
					'title' => 'Add Text For Second Option'
				),

			);
   		 }
		add_filter('cs_metabox_options', 'name_theme_metabox');
এখন সবগুলো ফিল্ড আলাদা ডাটা অ্যাড করবো। ব্যাকএন্ডের কাজ শেষ।


ডাটাগুলোকে একটা ভ্যারিয়েবল এ নিয়ে নিবো।
	<?php
    $my_variable = cs_get_option('our_field_group');
    
`foreach` লোপ চালিয়ে ডেটাগুলোকে আলাদা করবো। এবং ফ্রন্টেন্ডে অ্যাড করে দেবো।
	<?php
    $my_variable = cs_get_option('our_field_group');

	foreach($my_variable as $new_varible) {
	?>
	HTML Elements Here
	<?php } ?>

এবার ভ্যারিয়েবল এর মাধ্যমে ডাটাগুলো এন্ট্রি করবো।

	<?php
    $my_variable = cs_get_option('our_field_group');

	foreach($my_variable as $new_varible) {
	?>
	<h1><?php cs_get_option('option_id'); ?> </h1>
	<img src="<?php esc_url(wp_get_attachment_image_src( cs_get_option($new_varible['option_id']), 'thumbnail'[0]));?>">
	<?php } ?>
> এখানে সেই গ্রুপ ডাটাগুলোই শো করবে। যেগুলো আমরা গ্রুপে অ্যাড করেছি। ৩ টা হলে ৩টা। ৫ টা হলে ৫ টা অর্থাৎ যতোটা ইচ্ছা।

#### Create Accordion: ####
accordion হচ্ছে toggle collapse অপশন গ্রুপ। খুব সহজেই আমরা এটা করে ফেলতে পারি। তার জন্য আমাদের `fields` এর যায়গায় `sections` লিখতে হবে। এবং প্রতিটি অপশনের `$options[] = ` অংশটুকু কেটে শুধু অ্যারে রাখবো। অ্যারেগুলো অবশ্যই নেস্টেড হতে হবে অর্থাৎ কমা দিয়ে শেষ করতে হবে। সেমিকোলন দিলে error আসবে। যেহেতু সবগুলো একটা accordion


    name_theme_metabox($options){
        $options = array();

		$options[] = array(
			'name' => 'section_name', //name should be unique.
			'title' => 'Our Section',
			'icon' => 'fa fa-icon',
			'sections' => array(
				//start first section
				array(
				'id' => 'our_field_group',
				'type' => 'group', //must be
				'title' => 'Add Text For Second Option'
				'button_title' => 'Add button', //must be
				'accordion_title' => 'Add new field' //must be
				),
				'fields' => array(
					array(
					'id' => 'option_id',
					'type' => 'text',
					'title' => 'Add Text For First Option'
					),

					array(
					'id' => 'another_id',
					'type' => 'text',
					'title' => 'Add Text For Second Option'
				),
				//end first section

				//start second section				

				array(
				'id' => 'our_field_group',
				'type' => 'group', //must be
				'title' => 'Add Text For Second Option'
				'button_title' => 'Add button', //must be
				'accordion_title' => 'Add new field' //must be
				),
				'fields' => array(
					array(
					'id' => 'option_id',
					'type' => 'text',
					'title' => 'Add Text For First Option'
					),

					array(
					'id' => 'another_id',
					'type' => 'text',
					'title' => 'Add Text For Second Option'
				),
				//end second section		

			);
   		 }
		add_filter('cs_metabox_options', 'name_theme_metabox');

ব্যস কাজ শেষ।


----------


### CMB2: ###
কাস্টম মেটাবক্স তৈরি করার জন্য CMB2 ফ্রেমওয়ার্কটি ব্যবহার করা হয় সেটা কারো অজানা নয়। ধাপে ধাপে CMB2 এর কাজগুলো দেখবো।

কাজ করার প্রসেস প্রায় একই অন্য সব ফ্রেমওয়ার্ক গুলোর মতো। প্রথমে আমরা CMB2 ডাউনলোড করবো। এবং আমাদের থিম ডিরেক্টরিতে ইনক্লুড করবো। এবং CMB2 এর মেইন ফাইল যেটা অর্থাৎ `init.php` আমাদের `functions.php` তে অ্যাড করবো।

প্রথমেই আমরা ফ্রেমওয়ার্কটি ডাউনলোড করবো। তাদের অফিশিয়াল গিট রিপোজিটরি থেকে।

প্রথমেই আমাদের ডাউনলোড করা zip ফাইলটিকে এক্সট্রাক্ট করবো। এবং আমাদের ওয়ার্ডপ্রেস থিম ডিরেক্টরিতে একটা ফোল্ডার ফোল্ডার ইনক্লুড করবো। এবং এক্সট্রাকটেড ফাইলগুলো ঐ ফোল্ডারে মুভ করবো।

আমাদের ফ্রেমওয়ার্কের প্রায় সকল ফাইল ইনক্লুডেড হয়ে গেছে। এখন মেইন যে ফাইল অর্থাৎ `init.php` কে `functions.php` তে লিঙ্ক করবো। যাতে আমাদের থিম ফাংশনের সাথে অ্যাড হয়ে যায়।

    require get_template_directory() . '/inc/CMB2/init.php';

এখন রিলোড দিলে দেখতে পারবেন ফ্রেমওয়ার্কটি আমাদের থিমের সাথে অ্যাড হয়ে গেছে। আমাদের ইন্ট্রিগেশন পার্ট শেষ।

এখন আমরা নতুম একটা ফাইল ক্রিয়েট করে আমাদের নিজের মতো করে কাজ করবো। এজন্য `CMB2-configure.php` নামে একটা ফাইল ক্রিয়েট করছি। অথবা যেকোনো নামে ক্রিয়েট করতে পারেন।

তো আমরা যে নতুম ফাইলটি ক্রিয়েট করলাম এখন এটাকেও আমাদের `functions.php` তে কল করতে হবে।

    require get_template_directory() . '/inc/CMB2/CMB2-configure.php';

`CMB2-configure.php` তে এখন একটা ফাংশন ক্রিয়েট করবো। যেখানে আমরা সমস্ত কাজ করবো।

    name_cmb2(){
    
    }
> name এর যায়গায় আপনার থিম নেম প্রিফিক্স হিসেবে হবে।


এবার CMB2 এর অপশন এর একশন হুকটি আমরা ইউজ করবো।

    name_cmb2(){
    
    }
    add_action('cmb2_init', 'name_cmb2');

এখন বক্স নেবো। যেটাতে আমাদের মেটাডাটাগুলো স্টোর করবো।


    name_cmb2(){
    	new_cmb2_box(
			
		);
    }
    add_action('cmb2_init', 'name_cmb2');

এবার অ্যারে নিয়ে বক্সে ইনফরমেশন স্টোর করবো।


    name_cmb2(){
    	new_cmb2_box( array(
			'title' => 'my box',
			'name' => 'box name',
			'id' => 'id_name',
			'object_types' => array('custom_post_type_id') // you can add page or post
		));
    }
    add_action('cmb2_init', 'name_cmb2');

এখন আমরা আমাদের বক্সে ফিল্ড অ্যাড করবো। তো প্রথমেই বক্সটাকে একটা ভ্যারিয়েবলে নিয়ে যাই।


    name_cmb2(){
    	$myvar = new_cmb2_box( array(
			'title' => 'my box',
			'id' => 'id_name',
			'object_types' => array('custom_post_type_id') // you can add page or post
		));
    }
    add_action('cmb2_init', 'name_cmb2');

এখন একটা অবজেক্ট তৈরি করবো। এবং এর মাধ্যমে ফিল্ড অ্যাড করবো।


    $myvar -> add_fields( array(
		'name' => 'Title 1', //label
		'id' => 'id_name', //unique id name
		'type' => 'text',
    ));
    

এবার আমরা ডাটা ইনফরমেশনগুলোকে ফ্রন্টেন্ডে নিয়ে আসবো।

    <?php while(have_posts()) : the_post(); ?>
    //Our Design Content Here
    <?php endwhile; ?>
    
এখন আমাদের শুধু রেগুলার পোষ্ট শো করানোর কাজটা শেষ হলো। কিন্ত এই পোষ্ট টাইপের পোষ্ট দেখাতে একটা অবজেক্ট তৈরি করতে হবে। যার মাধ্যমে ওয়ার্ডপ্রেস এর ক্লাস থেকে ডাটা কল করবো।

    $name = new WP_Query();

> new কিওয়ার্ড দিয়ে অবজেক্ট তৈরি করেছি। যেটা $name variable এ স্টোর করেছি। তো এখন থেকে $name অবজেক্ট হিসেবে কাজ করবে।

এখন কোন পোষ্ট টাইপের পোষ্ট দেখাবে। এটা তাকে বুঝিয়ে দিতে হবে। মাল্টিপল ভ্যালু হতে পারে। এজন্য একটা অ্যারে তৈরি করবো।

    $name = new WP_Query(array(
    			'post_type' => 'name',
    ));
> এখানে name হচ্ছে আমাদের রেজিষ্টার করা পোষ্ট টাইপ নেম।

এবার `get_post_meta()` ফাংশনের মাধ্যমে আমাদের মেটাডাটাগুলোকে কল করবো। এবং `get_the_id()` দিয়ে পোষ্ট আইডি কল করবো।

    <?php echo get_post_meta(get_the_id(), 'id_name', true); ?>
    

> এখানে প্রথম প্যারামিটার ID পাওয়ার জন্য। দ্বিতীয় প্যারামিটার আমাদের মেটাবক্স আইডি। তৃতীয় প্যারামিটার আমাদের মেটা কি সিঙ্গেল ভ্যালু রিটার্ন করবে কি না এটা ডিফাইন করার জন্য।

এখন দেখবেন সবকিছুই ঠিকঠাক লোড হচ্ছে। আমাদের কাজ শেষ।

---

**প্রতিনিয়ত নতুন নতুন লেসন যুক্ত হবে এবং আপডেট হবে।***