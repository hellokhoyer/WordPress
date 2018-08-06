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
**প্রতিনিয়ত নতুন নতুন লেসন যুক্ত হবে এবং আপডেট হবে।***