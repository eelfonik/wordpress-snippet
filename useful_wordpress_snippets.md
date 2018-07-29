## Useful Wordpress snippets
## functions.php
#### remove admin bar in front-end for logged in user
```php
show_admin_bar( false );
```

#### add body class
```php
add_filter( 'body_class', function( $classes ) {
   return array_merge( $classes, array( 'class-name' ) );
} );
```

#### cache bust according to file modification time
```php
wp_enqueue_style( 'stylesheet', get_stylesheet_uri(), array(), filemtime( get_stylesheet_directory() . '/style.css' ) );
wp_enqueue_script( 'main', get_template_directory_uri() . '/js/main-min.js', array('jquery'), filemtime( get_template_directory() . '/js/main-min.js' ), true );
```

#### override default gallery output

```php
function fix_my_gallery($output, $attr) {
    global $post;

    static $instance = 0;
    $instance++;


    /**
     *  will remove this since we don't want an endless loop going on here
     */
    // Allow plugins/themes to override the default gallery template.
    //$output = apply_filters('post_gallery', '', $attr);

    // We're trusting author input, so let's at least make sure it looks like a valid orderby statement
    if ( isset( $attr['orderby'] ) ) {
        $attr['orderby'] = sanitize_sql_orderby( $attr['orderby'] );
        if ( !$attr['orderby'] )
            unset( $attr['orderby'] );
    }

    extract(shortcode_atts(array(
        'order'      => 'ASC',
        'orderby'    => 'menu_order ID',
        'id'         => $post->ID,
        'itemtag'    => 'div',
        'icontag'    => 'dt',
        'captiontag' => 'dd',
        'columns'    => 0,
        'size'       => 'full',
        'include'    => '',
        'exclude'    => ''
    ), $attr));

    $id = intval($id);
    if ( 'RAND' == $order )
        $orderby = 'none';

    if ( !empty($include) ) {
        $include = preg_replace( '/[^0-9,]+/', '', $include );
        $_attachments = get_posts( array('include' => $include, 'post_status' => 'inherit', 'post_type' => 'attachment', 'post_mime_type' => 'image', 'order' => $order, 'orderby' => $orderby) );

        $attachments = array();
        foreach ( $_attachments as $key => $val ) {
            $attachments[$val->ID] = $_attachments[$key];
        }
    } elseif ( !empty($exclude) ) {
        $exclude = preg_replace( '/[^0-9,]+/', '', $exclude );
        $attachments = get_children( array('post_parent' => $id, 'exclude' => $exclude, 'post_status' => 'inherit', 'post_type' => 'attachment', 'post_mime_type' => 'image', 'order' => $order, 'orderby' => $orderby) );
    } else {
        $attachments = get_children( array('post_parent' => $id, 'post_status' => 'inherit', 'post_type' => 'attachment', 'post_mime_type' => 'image', 'order' => $order, 'orderby' => $orderby) );
    }

    if ( empty($attachments) )
        return '';

    if ( is_feed() ) {
        $output = "\n";
        foreach ( $attachments as $att_id => $attachment )
            $output .= wp_get_attachment_link($att_id, $size, true) . "\n";
        return $output;
    }

    $itemtag = tag_escape($itemtag);
    $captiontag = tag_escape($captiontag);
    $columns = intval($columns);
    $itemwidth = $columns > 0 ? floor(100/$columns) : 100;
    $float = is_rtl() ? 'right' : 'left';

    $selector = "gallery-{$instance}";

    $gallery_style = $gallery_div = '';
    if ( apply_filters( 'use_default_gallery_style', true ) )
        /**
         * this is the css you want to remove
         *  #1 in question
         */
        /*
        $gallery_style = "
        <style type='text/css'>
            #{$selector} {
                margin: auto;
            }
            #{$selector} .gallery-item {
                float: {$float};
                margin-top: 10px;
                text-align: center;
                width: {$itemwidth}%;
            }
            #{$selector} img {
                border: 2px solid #cfcfcf;
            }
            #{$selector} .gallery-caption {
                margin-left: 0;
            }
        </style>
        <!-- see gallery_shortcode() in wp-includes/media.php -->";
        */
    $size_class = sanitize_html_class( $size );
    $gallery_div = "<div id='$selector' class='gallery galleryid-{$id} gallery-columns-{$columns} gallery-size-{$size_class}'>";
    $output = apply_filters( 'gallery_style', $gallery_style . "\n\t\t" . $gallery_div );

    $i = 0;
    foreach ( $attachments as $id => $attachment ) {
        $image = isset($attr['link']) && 'file' == $attr['link'] ? wp_get_attachment_image($id, $size, false) : wp_get_attachment_image($id, $size, false);

        $output .= "
            <{$itemtag} class='gallery-item'>
                $image
            </{$itemtag}>";
        /*
         * This is the caption part so i'll comment that out
         * #2 in question
         */
        /*
        if ( $captiontag && trim($attachment->post_excerpt) ) {
            $output .= "
                <{$captiontag} class='wp-caption-text gallery-caption'>
                " . wptexturize($attachment->post_excerpt) . "
                </{$captiontag}>";
        }*/
        if ( $columns > 0 && ++$i % $columns == 0 )
            $output .= '<br style="clear: both" />';
    }

    /**
     * this is the extra br you want to remove so we change it to jus closing div tag
     * #3 in question
     */
    /*$output .= "
            <br style='clear: both;' />
        </div>\n";
     */

    $output .= "</div>\n";
    return $output;
}
add_filter("post_gallery", "fix_my_gallery",10,2);
```
#### remove default images upload sizes
```php
function remove_default_image_sizes( $sizes) {
    unset( $sizes['thumbnail']);
    unset( $sizes['medium']);
    unset( $sizes['large']);
     
    return $sizes;
    }
add_filter('intermediate_image_sizes_advanced', 'remove_default_image_sizes');
```
#### add taxonomies to pages

```php
function add_taxonomies_to_pages() {  
		// Add tag metabox to page
		register_taxonomy_for_object_type('post_tag', 'page'); 
		// Add category metabox to page
		register_taxonomy_for_object_type( 'category', 'page' );
}
add_action( 'init', 'add_taxonomies_to_pages' );
```

#### add svg support for media upload

```php
function cc_mime_types($mimes) {
	  $mimes['svg'] = 'image/svg+xml';
	  return $mimes;
}
add_filter('upload_mimes', 'cc_mime_types');
```

#### remove p tags around uploaded images

```php
function filter_ptags_on_images($content){
	   return preg_replace('/<p>\s*(<a .*>)?\s*(<img .* \/>)\s*(<\/a>)?\s*<\/p>/iU', '\1\2\3', $content);
}

add_filter('the_content', 'filter_ptags_on_images');
```

#### remove default link to images

```php
add_filter( 'the_content', 'attachment_image_link_remove_filter' );
function attachment_image_link_remove_filter( $content ) {
 $content =
 preg_replace(
 array('{<a(.*?)(wp-att|wp-content/uploads)[^>]*><img}',
 '{ wp-image-[0-9]*" /></a>}'),
 array('<img','" />'),
 $content
 );
 return $content;
 } 
```
 
#### using lazy loading by replace the generated the_content() images `src` with `data-src`
```php
function add_lazyload($content) {

    $content = mb_convert_encoding($content, 'HTML-ENTITIES', "UTF-8");
    $dom = new DOMDocument();
    @$dom->loadHTML($content);

    foreach ($dom->getElementsByTagName('img') as $node) {  
        $oldsrc = $node->getAttribute('src');
        $node->setAttribute("data-src", $oldsrc );
        // $newsrc = ''.get_template_directory_uri().'/images/nothing.gif';
        $newsrc = '';
        $node->setAttribute("src", $newsrc);
    }
    $newHtml = preg_replace('/^<!DOCTYPE.+?>/', '', str_replace( array('<html>', '</html>', '<body>', '</body>'), array('', '', '', ''), $dom->saveHTML()));
    return $newHtml;
    }
    add_filter('the_content', 'add_lazyload');
```
#### Enable support for Post Formats. ([link](https://developer.wordpress.org/themes/functionality/post-formats/))

```php
add_theme_support( 'post-formats', array(
		'image',
		'video',
	) );
```

#### create custom post type

```php
add_action( 'init', 'create_post_type' );
function create_post_type() {
	register_post_type( 'projets',
	    array(
	      'labels' => array(
	        'name' => __( 'Projets' ),
	        'singular_name' => __( 'Projet' ),
	        'all_items' => 'All Projets',
	        'add_new_item' => 'Add new projet',
	        'not_found' => 'No projets found',
	        'featured_image' => 'Chinese character svg',
	        'set_featured_image' => 'Set Chinese svg',
	        'remove_featured_image' => 'Remove Chinese svg',
	        'use_featured_image' => 'Use as title chinese svg'
	      ),
	      'public' => true,
	      'has_archive' => true,
	      'supports' => array( 'title', 'thumbnail', 'page-attributes','revisions' ),
	      'rewrite' => array('slug' => 'projets'),
	      'menu_position' => 5,
	      'hierarchical'=>true,
	      'taxonomies' => array('category'),
	    )
	 );
  	register_post_type( 'projet_slides',
	    array(
	      'labels' => array(
	        'name' => __( 'Projet Slides' ),
	        'singular_name' => __( 'Projet Slide' ),
	        'not_found' => 'No projet slides found',
            'insert_into_item' => 'Insert into slide',
	      ),
	      'public' => true,
	      'has_archive' => true,
	      'supports' => array( 'title', 'editor', 'page-attributes','post-formats','revisions' ),
	      'rewrite' => array('slug' => 'projets/%projets_name%', "with_front" => true),
	      'menu_position' => 5,
	    )
  	);
}
```

**One note on CTP:** if it's ```hierarchical => false```, then it will be treated like default *post* by wordpress core and most plugins, if it's ```hierarchical => true```, then it'll act like default *page*. 

#### link a custom post type as child to another custom post type (or any other post/page)

- this one the child CPT **should** have `hierarchical => false` (default is false when creating custom post type):

```php
add_action('add_meta_boxes', function() {
    add_meta_box('children-post-type-name-parent', 'Parents', 'slides_attributes_meta_box', 'children', 'side', 'default');
});

function slides_attributes_meta_box($post) {
        $pages = wp_dropdown_pages(array(
        	'post_type' => 'projets', 
        	'selected' => $post->post_parent, 
        	'name' => 'parent_id', 
        	'show_option_none' => __('(no parent)'), 
        	'sort_column'=> 'menu_order, post_title', 
        	'echo' => 0));
        if ( ! empty($pages) ) {
            echo $pages;
        } // end empty pages check
}
```

- another solution which **required** child CPT to have `hierarchical => true` (personally prefer this one)

```php
add_action('admin_menu', function(){ 
	remove_meta_box('pageparentdiv', 'children', 'normal');
});

add_action('add_meta_boxes', function() { 
	add_meta_box('children-post-type-name-parent', 'Parents', 'children_attributes_meta_box', 'children', 'side', 'high');
});

function children_attributes_meta_box($post) {
    $post_type_object = get_post_type_object($post->post_type);
    if ( $post_type_object->hierarchical ) {
      $pages = wp_dropdown_pages(array(
      	'post_type' => 'part', 
      	'selected' => $post->post_parent, 
      	'name' => 'parent_id', 
      	'show_option_none' => __('(no parent)'), 
      	'sort_column'=> 'menu_order, post_title', 
      	'echo' => 0));
      if ( ! empty($pages) ) {
        echo $pages;
      } // end empty pages check
    } // end hierarchical check.
  }
```

- **the rewrite permalink rule**

```php
add_action( 'init', function() {
    add_rewrite_rule( '^parents/(.*)/([^/]+)/?$','index.php?children=$matches[2]','top' );
});

add_filter( 'post_type_link', function( $link, $post ) {
    if ( 'children' == get_post_type( $post ) ) {
        //Lets go to get the parent post name
        if( $post->post_parent ) {
            $parent = get_post( $post->post_parent );
            if( !empty($parent->post_name) ) {
                return str_replace( '%parents_name%', $parent->post_name, $link );
            //here '%parents_name%' represents the rewrite rule for slug in child CTP
            }
        } else {
            //This seems to not work. It is intented to build pretty permalinks
            //when episodes has not parent, but it seems that it would need
            //additional rewrite rules
            //return str_replace( '/%parents_name%', '', $link );
        }

    }

    return $link;
}, 10, 2 );
```

#### Add any kind of meta-box to any post type

A more detailed [instruction](https://www.sitepoint.com/adding-meta-boxes-post-types-wordpress/) about how to add **any kind of meta-box** to **any post type**, and eventually use the input value to `update_post_meta` or `wp_set_object_terms`, then retrieve them afterwards on post load.

#### Creating single select wordpress taxonomies
A great article [here](http://sudarmuthu.com/blog/creating-single-select-wordpress-taxonomies/)!!

- **step 1**: 
inside `functions.php`, after create a custom taxonomy, we can create a single radio button select: 

```php
function movie_rating_meta_box( $post ) {
	$terms = get_terms( 'movie_rating', array( 'hide_empty' => false ) );
	$post  = get_post();
	$rating = wp_get_object_terms( $post->ID, 'movie_rating', array( 'orderby' => 'term_id', 'order' => 'ASC' ) );
	$name  = '';
    if ( ! is_wp_error( $rating ) ) {
    	if ( isset( $rating[0] ) && isset( $rating[0]->name ) ) {
			$name = $rating[0]->name;
	    }
    }
	foreach ( $terms as $term ) {
?>
		<label title='<?php esc_attr_e( $term->name ); ?>'>
		    <input type="radio" name="movie_rating" value="<?php esc_attr_e( $term->name ); ?>" <?php checked( $term->name, $name ); ?>>
			<span><?php esc_html_e( $term->name ); ?></span>
		</label><br>
<?php
    }
}
```
So this will place a radio select on any post type with **THIS** taxonomy.

- **step 2**: 
send the value back to database inside the related post `terms`

```php
/**
 * Save the movie meta box results.
 *
 * @param int $post_id The ID of the post that's being saved.
 */
function save_movie_rating_meta_box( $post_id ) {
	if ( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE ) {
		return;
	}
	if ( ! isset( $_POST['movie_rating'] ) ) {
		return;
	}
	$rating = sanitize_text_field( $_POST['movie_rating'] );
	
	// A valid rating is required, so don't let this get published without one
	if ( empty( $rating ) ) {
		// unhook this function so it doesn't loop infinitely
		remove_action( 'save_post_movie', 'save_movie_rating_meta_box' );
		$postdata = array(
			'ID'          => $post_id,
			'post_status' => 'draft',
		);
		wp_update_post( $postdata );
	} else {
		$term = get_term_by( 'name', $rating, 'movie_rating' );
		if ( ! empty( $term ) && ! is_wp_error( $term ) ) {
			wp_set_object_terms( $post_id, $term->term_id, 'movie_rating', false );
		}
	}
}
add_action( 'save_post_movie', 'save_movie_rating_meta_box' );
```
**N.B.** the action `save_post_movie` is a best practice to avoid unnecessary query when saving post, but only do this action upon a particular post type, so it means `save_post_{post_type}`

- **step 3** (optional) :
if the selection is mandatory for save the post, we could show an error to user

```php
/**
 * Display an error message at the top of the post edit screen explaining that ratings is required.
 *
 * Doing this prevents users from getting confused when their new posts aren't published, as we
 * require a valid rating custom taxonomy.
 *
 * @param WP_Post The current post object.
 */
function show_required_field_error_msg( $post ) {
	if ( 'movie' === get_post_type( $post ) && 'auto-draft' !== get_post_status( $post ) ) {
	    $rating = wp_get_object_terms( $post->ID, 'movie_rating', array( 'orderby' => 'term_id', 'order' => 'ASC' ) );
        if ( is_wp_error( $rating ) || empty( $rating ) ) {
			printf(
				'<div class="error below-h2"><p>%s</p></div>',
				esc_html__( 'Rating is mandatory for creating a new movie post' )
			);
		}
	}
}
// Unfortunately, 'admin_notices' puts this too high on the edit screen
add_action( 'edit_form_top', 'show_required_field_error_msg' );
```

#### Add template selection to CPT
check this [answer](https://wordpress.stackexchange.com/a/204671)

```php
/** Custom Post Type Template Selector **/
function cpt_add_meta_boxes() {
    $post_types = get_post_types();
    foreach( $post_types as $ptype ) {
        if ( $ptype !== 'page') {
            add_meta_box( 'cpt-selector', 'Attributes', 'cpt_meta_box', $ptype, 'side', 'core' );
        }
    }
}
add_action( 'add_meta_boxes', 'cpt_add_meta_boxes' );

function cpt_remove_meta_boxes() {
    $post_types = get_post_types();
    foreach( $post_types as $ptype ) {
        if ( $ptype !== 'page') {
            remove_meta_box( 'pageparentdiv', $ptype, 'normal' );
        }
    }
}
add_action( 'admin_menu' , 'cpt_remove_meta_boxes' );

function cpt_meta_box( $post ) {
    $post_meta = get_post_meta( $post->ID );
    $templates = wp_get_theme()->get_page_templates();

    $post_type_object = get_post_type_object($post->post_type);
    if ( $post_type_object->hierarchical ) {
        $dropdown_args = array(
            'post_type'        => $post->post_type,
            'exclude_tree'     => $post->ID,
            'selected'         => $post->post_parent,
            'name'             => 'parent_id',
            'show_option_none' => __('(no parent)'),
            'sort_column'      => 'menu_order, post_title',
            'echo'             => 0,
        );

        $dropdown_args = apply_filters( 'page_attributes_dropdown_pages_args', $dropdown_args, $post );
        $pages = wp_dropdown_pages( $dropdown_args );

        if ( $pages ) { 
            echo "<p><strong>Parent</strong></p>";
            echo "<label class=\"screen-reader-text\" for=\"parent_id\">Parent</label>";
            echo $pages;
        }
    }

    // Template Selector
    echo "<p><strong>Template</strong></p>";
    echo "<select id=\"cpt-selector\" name=\"_wp_page_template\"><option value=\"default\">Default Template</option>";
    foreach ( $templates as $template_filename => $template_name ) {
        if ( $post->post_type == strstr( $template_filename, '-', true) ) {
            if ( isset($post_meta['_wp_page_template'][0]) && ($post_meta['_wp_page_template'][0] == $template_filename) ) {
                echo "<option value=\"$template_filename\" selected=\"selected\">$template_name</option>";
            } else {
                echo "<option value=\"$template_filename\">$template_name</option>";
            }
        }
    }
    echo "</select>";

    // Page order
    echo "<p><strong>Order</strong></p>";
    echo "<p><label class=\"screen-reader-text\" for=\"menu_order\">Order</label><input name=\"menu_order\" type=\"text\" size=\"4\" id=\"menu_order\" value=\"". esc_attr($post->menu_order) . "\" /></p>";
}

function save_cpt_template_meta_data( $post_id ) {

    if ( isset( $_REQUEST['_wp_page_template'] ) ) {
        update_post_meta( $post_id, '_wp_page_template', $_REQUEST['_wp_page_template'] );
    }
}
add_action( 'save_post' , 'save_cpt_template_meta_data' );

function custom_single_template($template) {
    global $post;

    $post_meta = ( $post ) ? get_post_meta( $post->ID ) : null;
    if ( isset($post_meta['_wp_page_template'][0]) && ( $post_meta['_wp_page_template'][0] != 'default' ) ) {
        $template = get_template_directory() . '/' . $post_meta['_wp_page_template'][0];
    }

    return $template;
}
add_filter( 'single_template', 'custom_single_template' );
/** END Custom Post Type Template Selector **/
```

#### Enable the front page selection for CPT
see this [answer](https://wordpress.stackexchange.com/a/126271)
**SHOULD REWORK!!!** this snippet this too damaging! It will add hook to anything used `wp_dropdown_page`....

```php
function add_campaign_page_to_dropdown( $pages ){
    $args = array(
        'post_type' => 'campaign'
    );
    $items = get_posts($args);
    $pages = array_merge($pages, $items);

    return $pages;
}
add_filter( 'get_pages', 'add_campaign_page_to_dropdown' );

function enable_front_page_campaign( $query ){
    if('' == $query->query_vars['post_type'] && 0 != $query->query_vars['page_id'])
        $query->query_vars['post_type'] = array( 'page', 'campaign' );
}
add_action( 'pre_get_posts', 'enable_front_page_campaign' );
```

## navigation (usually header.php)

- Add filters using any taxonomy to filter contents (normally a list of post/page titles, thumbnails ) **on the page**, since the default taxonomy menu will directly go to category or tag page.

**the filter**

```php
<ul id="filters">
	<?php 
	$terms = get_terms("category"); // get all categories, but you can use any taxonomy
	$count = count($terms); //How many are they?
	if ( $count > 0 ){  //If there are more than 0 terms
		foreach ( $terms as $term ) {  //for each term:
			echo "<li><a href='#' data-filter='.".$term->slug."'>" . $term->name . "</a></li>\n";//create a list item with the current term slug for sorting, and name for label
		}
	} 
	?>
</ul>
```

**the listed content**


```php
<?php 
			$args = array(
				'post_type' =>'projets',
				'posts_per_page'=> -1,
				'orderby' => 'menu_order',
				'order'   => 'ASC',
			);
			$the_query = new WP_Query($args);?>
			<?php if ( $the_query->have_posts() ) : ?>
			    <ul id="list">
			    <?php while ( $the_query->have_posts() ) : $the_query->the_post(); 
				$termsArray = get_the_terms( $post->ID, "category" ); 
				//Get the terms for this particular item
				$termsString = ""; 
				//initialize the string that will contain the terms
					foreach ( $termsArray as $term ) { // for each term 
						$termsString .= $term->slug.' '; //create a string that has all the slugs 
					}
				?> 
				<li class="<?php echo $termsString;?> projet"><?php // 'projet' is used as an identifier ?>
				    <a href="<?php echo get_page_link(); ?>" class="page-id-<?php echo get_the_ID(); ?>">
				    	<?php if ( has_post_thumbnail() )
				    	{the_post_thumbnail();}?>
				    	<span><?php the_title();?></span>
				    </a>
				</li> <!-- end item -->
			    <?php endwhile;  ?>
			    </ul> <!-- end list -->
			<?php endif;
		wp_reset_postdata();
?>
```
 

## Some custom post type template
- infinite **page** navigation

```php
<?php
	/**
	*  Infinite next and previous page looping in WordPress
	 */


		$pagelist = get_pages('posts_per_page=-1&post_type=projets&sort_column=menu_order&sort_order=asc');
		$pages = array();
		foreach ($pagelist as $page) {
		   $pages[] += $page->ID;
		}

		$current = array_search(get_the_ID(), $pages);
		$prevID = $pages[$current-1];
		$nextID = $pages[$current+1];
		$lastID = end($pages);
		$firstID = $pages[0];
		?>

			<div class="prev-projet projet-nav">
				<?php if (!empty($prevID)) { ?>
				<a href="<?php echo get_permalink($prevID); ?>"
				  title="<?php echo get_the_title($prevID); ?>">Previous</a>
				<?php } else { ?>
				<a href="<?php echo get_permalink($lastID); ?>">Go last</a>
				<?php } ?>
			</div>
			<div class="next-projet projet-nav">
				<?php if (!empty($nextID)) { ?>
				<a href="<?php echo get_permalink($nextID); ?>" 
				 title="<?php echo get_the_title($nextID); ?>">Next</a>
				<?php } else { ?>
				<a href="<?php echo get_permalink($firstID); ?>">Go first</a>
				<?php } ?>
			</div>


		<?php  // End of the loop.
		?>
```

- or infinite **post** navigation

```php
<?php 
/**
 *  Infinite next and previous post looping in WordPress
 */
if( get_adjacent_post(false, '', true) ) { 
	previous_post_link('%link', '&larr; Previous Post');
} else { 
    $first = new WP_Query('posts_per_page=1&order=DESC'); $first->the_post();
    	echo '<a href="' . get_permalink() . '">&larr; Previous Post</a>';
  	wp_reset_query();
}; 
    
if( get_adjacent_post(false, '', false) ) { 
	next_post_link('%link', 'Next Post &rarr;');
} else { 
	$last = new WP_Query('posts_per_page=1&order=ASC'); $last->the_post();
    	echo '<a href="' . get_permalink() . '">Next Post &rarr;</a>';
    wp_reset_query();
};  ?>
```



## All about wp_query

see link [here](https://code.tutsplus.com/tutorials/wp_query-arguments-taxonomies--cms-23090) for detailed **tanomoxies**.

And [this](https://premium.wpmudev.org/blog/mastering-wp-query/?utm_expid=3606929-94.Ie3dH-CaRwe6MU3VrZsdvw.0&utm_referrer=https%3A%2F%2Fwww.google.fr%2F) exellent guide to **using wp_query in general**.

**note**: when getting properties from wp_query, use `$your_query->found_posts`, not `$your_query->$found_posts`.

- How to find **position of post** inside a wp_query: 
see [link](http://wordpress.stackexchange.com/a/146111)

```php
$query = new WP_Query( array(
    'TAXONOMY' => 'TAXONOMY TERM', 
    'posts_per_page' => -1, 
    'post_status' => 'publish',
    'orderby' => 'date', // be sure posts are ordered by date
    'order' => 'ASC', // be sure order is ascending
    'fields' => 'ids' // get only post ids
));

global $post; // current post object

$i = array_search( $post->ID, $query->posts ) + 1; // add 1 because array are 0-based

echo "Post {$i} / {$query->post_count} in TAXONOMY TERM";
```



## Ajax pagination
- **pagination** combined with **sort/filter/search** in **Ajax**, great code piece [here](http://carlofontanos.com/wordpress-front-end-ajax-pagination-with-search-and-sort/).