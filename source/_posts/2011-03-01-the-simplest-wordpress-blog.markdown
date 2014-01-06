---
comments: true
date: 2011-03-01 21:28:14
layout: post
slug: the-simplest-wordpress-blog
title: The Simplest WordPress Blog
categories:
- php
- wordpress
---

So I like to start all my Wordpress theme designs with a very simplistic Wordpress base and then add all the complicated functionality found in my themes. I think it is a useful learning tool for the beginner to get used to Wordpress programming. So here's my guide for setting up a really simple blog.

<!-- more -->


First of all go to your wordpress root and find the folder wp-content/themes/ and create a new folder with the name of the theme you'd like. Then set up the meta for the theme. This includes setting up the theme name, description and author, this is done by writing a few details in a new file that you need to create called style.css. At the top of the file put some of the details for your theme, you can replace the details that I have entered for your own:

{% codeblock lang:css %}
/*
theme name: basic
theme uri:
description: a starter theme
version: 0.1
author: thomas whitton
author uri: http://www.thomaswhitton.com
tags: simple, clean, single column
*/
{% endcodeblock %}

We're not going to be putting anything in the css file, but you may want to put in resets for the most commonly used attributes at the beginning, there are loads of these css code segments on the internet and you just need to to pick one and slightly alter it for you particular site, search for style reset in google :).

Then we'll add the index.php, the glue that holds the rest of the files together. First of all we'll create the main Wordpress loop, as this is the most important part of the design. This is the main part of your site, generally on the front page, that displays a selection of your posts.

{% codeblock lang:php %}
<?php if (have_posts()) : ?>;
	<?php while (have_posts()) : the_post(); ?>
<?php endwhile; ?>
<?php else : ?>
	<h2>Not Found</h2>
	<p>Sorry, but you are looking for something that isn't here.</p>
<?php endif; ?>
{% endcodeblock %}

Now this code is just checking if any posts exist and then looping through all the posts in reverse chronilogical order. Now inside the loop nothing is happening so whats the point? Well now we can add some special wordpress functions that will extract the information from each post that we loop through. The code also supplies an error message if there are no posts to display.

{% codeblock lang:php %}
<?php the_permalink() ?> // Returns the url of the permalink to the post
<?php the_title(); ?> // Returns the title of the post
<?php the_author(); ?> // Returns the author of the post
<?php // Returns the content of the post ?>
<?php the_content('Read the rest of this entry &raquo;'); ?>
{% endcodeblock %}

These functions are simple enough and are easily used within the loop shown previously, the only slight different is with the_content() function which takes a string as a parameter, this just provides the words for the more text link.

For showing how many comments a post has, still inside the loop, we have a slightly more complicated bit of code. First we have to check certain conditions:

{% codeblock lang:php %}
//would normal include different styles for comment closed/open status here
<?php if ('closed' == $post->comment_status) : ?>
<?php else : ?>
<?php endif; ?>
<?php comments_popup_link('leave a comment',
'1 comment', '% comments', '', 'comments closed'); ?>
{% endcodeblock %}

Here we check if the post allows for comments first, create a div for each case and then use the comments_popup_link() function with parameters showing what words we'd like to show for each case particular case of comment status. It's pretty easy to work out what each parameter corresponds to from the example given above.

So thats it for the loop, we obviously have to add a bit more divs and css styling to get everything formatted in the correct way but that goes beyond the scope of this tutorial.

One further bit of code to add after the end of the loop but still inside the if(have_post()) function is the code for pagination. Again wordpress makes this simple and is just a matter of adding:

{% codeblock lang:php %}
<?php next_posts_link('&laquo; Older') ?>
<?php previous_posts_link('Newer &raquo;') ?>
{% endcodeblock %}

We then have to include the header, sidebar and footer pages which we'll quickly create in a minute. This is easily done using the built in Wordpress functions as follows. Adding this function to the top of the index.php file:

{% codeblock lang:php %}
<?php get_header(); ?>
{% endcodeblock %}

And then to the bottom of index.php:

{% codeblock lang:php %}
<?php get_sidebar(); ?>
<?php get_footer(); ?>
{% endcodeblock %}

These functions pull in the the header.php, sidebar.php and footer.php repectively, and we'll create these files now. First we'll create the footer.php as it is simpler. All we need to do is add this code to the file:

{% codeblock lang:php %}
<?php wp_footer(); ?>
</body>
</html>
{% endcodeblock %}

The wp_footer() function provides a hook to the footer of the page which Wordpress and Wordpress plugins can utilize. The rest of the file is just the closing of the html tags: body and html, which we will open shortly in the header.php file.

The sidebar.php gives the code for the sidebar on the site, this is quite a common wordpress blog feature. We'll check to see if the their are any widgets loaded in the primary widget area and if not we'll put our own in. Then we'll add a secondry widget area for less important widgets.

{% codeblock lang:php %}
<ul>
<?php
if (!dynamic_sidebar('primary-widget-area')) : ?>

	<li>
	<?php get_search_form(); ?>
	</li>

	<li>
	<h3>Archives</h3>
	<ul>
	<?php wp_get_archives('type=monthly'); ?>
	</ul>
	</li>

	<li>
	<h3>Meta</h3>
	<ul>
	<?php wp_register(); ?>
	<li><?php wp_loginout(); ?></li>
	<?php wp_meta(); ?>
	</ul>
	</li>

<?php endif;?>
</ul>

<?php if(is_active_sidebar('secondary-widget-area')) : ?>

	<ul>
	<?php dynamic_sidebar('secondary-widget-area'); ?>
	</ul>

<?php endif; ?>
{% endcodeblock %}

Now a lot of what is contained in the header.php file is the standard html head code which contains detailed information about your site, now Wordpress again makes this slightly easier by using specific functions, but really most of it is just a copy and paste job.

{% codeblock lang:php %}
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" <?php language_attributes(); ?>>

<head>
<meta http-equiv="Content-Type" content="
<?php bloginfo('html_type'); ?>; charset=
<?php bloginfo('charset'); ?>" />
<title>
<?php wp_title(':', true, 'right'); ?> <?php bloginfo('name'); ?>
</title>
<link rel="stylesheet" href="
<?php bloginfo('stylesheet_url'); ?>" type="text/css" media="screen" charset="utf-8" />
<link rel="alternate" type="application/rss+xml" title="
<?php bloginfo('name'); ?> RSS Feed" href="
<?php bloginfo('rss2_url'); ?>" />
<link rel="alternate" type="application/atom+xml" title="
<?php bloginfo('name'); ?> Atom Feed" href="
<?php bloginfo('atom_url'); ?>" />
<link rel="pingback" href="
<?php bloginfo('pingback_url'); ?>" />
<?php wp_head(); ?>
</head>

<body <?php body_class(); ?>>

<h1><a href="
<?php echo get_option('home'); ?>/" title="Home">
<?php bloginfo('name');?></a></h1>
{% endcodeblock %}

The only things to take note of are the rss and atom lines that will automatically generates rss and atom feeds using wordpress functions. Also the final line which creates your blog title and links it to your homepage. Apart from these it is pretty each to tell exacty what each wordpress function is doing.

Now as long as you have customized the look and format of your blog using some div's and css you have yourself a front page that shows off your latest posts. At the moment the hyperlinks associated with each post have not been implemented, this will be covered in the 2nd half of this tutorial where we will be looking at individual post pages, the comments for each post and the comment input form.

Please head on over to [Part 2](http://www.thomaswhitton.com/thomaswhitton/2011/03/06/the-simplest-wordpress-blog-part-2/).
