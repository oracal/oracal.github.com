---
comments: true
date: 2011-03-06 21:07:32
layout: post
slug: the-simplest-wordpress-blog-part-2
title: The Simplest Wordpress Blog, Part 2
wordpress_id: 139
categories:
- PHP
post_format:
- Standard
tags:
- PHP
- Wordpress
---

This is the second part of my simple wordpress tutorial, this will go through showing the blog post, the comments section and the search functionally. Putting this tutorial together with the last you'll have a completely unstylized functioning blog, which you can add all the styling you want.



<!-- more -->



So now onto single.php, this is the code to display how a single post is shown. First we'll look at the code to insert the post.



{% codeblock lang:php %}
<?php get_header(); ?>

<?php if (have_posts()) : while (have_posts()) : the_post(); ?>

	<h3><?php the_title(); ?></h3>
	<h4><?php the_author(); ?></h4>
	<?php the_content(); ?>

	<?php
	$year = get_the_time('Y');
	$month = get_the_time('m');
	$day = get_the_time('d');
	?>

	Published: <a href="<?php echo get_day_link("$year", "$month","$day"); ?>"><?php the_time('F j, Y'); ?></a>
	Filed Under: <?php the_category(', '); ?>
	<?php the_tags('Tags: ', ' : ', ''); ?>

	<?php comments_template(); ?>

	<?php previous_post_link('%link', '&laquo; Previous Post'); ?>
	<?php next_post_link('%link', 'Next Post &raquo;') ?>

<?php endwhile; else: ?>

	<p>Sorry, no posts matched your criteria.</p>

<?php endif; ?>

<?php get_sidebar(); ?>
<?php get_footer(); ?>
{% endcodeblock %}



We first set up the loop that checks for the current post using the following code:



{% codeblock lang:php %}
<?php if (have_posts()) : while (have_posts()) : the_post(); ?>
{% endcodeblock %}



We then extract the required information for the post using wordpress functions, the following commands are the most commonly used:

{% codeblock lang:php %}
<?php the_title(); // echo's the post title ?>
<?php the_author(); // echo's the post author ?>
<?php the_content(); // echo's the post content ?>
<?php the_time(); // echo's the date/time of the post ?>
<?php the_category(); // echo's the category of the post as well as a link to the respective category page ?>
<?php the_tags(); // echo's the tags of the post as well as a link to the respective tag page ?>
{% endcodeblock %}



Now the only other functions which we need to include in the single.php file and which we haven't seen already is the worpdress function which pulls in the comments.php file and the functions for linking to the next and previous posts:



{% codeblock lang:php %}
<?php comments_template(); ?>
<?php previous_post_link('%link', '&laquo; Previous Post'); ?>
<?php next_post_link('%link', 'Next Post &raquo;') ?>
{% endcodeblock %}



The entire single.php follows a very similar pattern to the index.php so you should have no real problem following it, the comments.php file is slightly more complicated and probably the most complicated part of the wordpress system, but they still manage to make it quite simple for us. I'm going to split the file comments.php up into managable chucks and explain each chunk, heres the first:



{% codeblock lang:php %}
<?php // Do not delete these lines
if (!empty($_SERVER['SCRIPT_FILENAME']) && 'comments.php' == basename($_SERVER['SCRIPT_FILENAME']))
	die ('Please do not load this page directly. Thanks!');

if ( post_password_required() ) { ?>
	<p>This post is password protected. Enter the password to view comments.</p>
	<?php
	return;
}
?>
{% endcodeblock %}



This is wordpress's own code, do not touch this. It pretty much stops people from accessing the page directly and checks to see if you need a password to view comments, not really something we should worry about, but make sure you include it.



{% codeblock lang:php %}
<?php if ( have_comments() ) : ?>
	<h3><?php comments_number('No Responses', 'One Response', '% Responses' );?> to "<?php the_title(); ?>"</h3>

	<?php previous_comments_link() ?>
	<?php next_comments_link() ?>
	<ol>
	<?php wp_list_comments('avatar_size=48'); ?>
	</ol>

	<?php previous_comments_link() ?>
	<?php next_comments_link() ?>
{% endcodeblock %}



This chunk of code tests whether there are comments in the wordpress query, and if there is, then the code first of all uses {% codeblock lang:php %}<?php comments_number('No Responses', 'One Response', '% Responses' );?>{% endcodeblock %} to print out a statement depending on how many comments there are, the parameters given to the function show quite nicely exactly what the function does and what each parameter does.





Next we provide a link to pages of comments for when pagination is used and then we output the comments using this handy WordPress function:



{% codeblock lang:php %}
<?php wp_list_comments('avatar_size=48'); ?>
{% endcodeblock %}



The parameter given here tells us what sixe to put the avatar images on the site, there are a few other parameters and you should probably check out on the wordpress codex, but for our simple blog we don't need it.





Next we have the chunk of code that will generally just display nothing, but you might want to put in a message saying that there are no comments or something like that here.



{% codeblock lang:php %}
<?php else : ?>

	<?php if ( comments_open() ) : ?>

  	 <?php else : ?>

	<?php endif; ?>
<?php endif; ?>
{% endcodeblock %}



This chunk outputs the form where comments are inputted, its quite long because it displays a slightly different form if the user is already logged in. Which is easily checked using:



{% codeblock lang:php %}
<?php if ( is_user_logged_in()) : ?>
{% endcodeblock %}



The rest of the code is pretty easy to follow:



{% codeblock lang:php %}
<?php if ( comments_open() ) : ?>

	<?php if ( get_option('comment_registration') && !is_user_logged_in() ) : ?>

	<p>You must be <a href="<?php echo get_option('siteurl'); ?>/wp-login.php?redirect_to=<?php the_permalink(); ?>">logged in</a> to post a comment.</p>

	<?php else : ?>

        	<form action="<?php echo get_option('siteurl'); ?>/wp-comments-post.php" method="post">

        	<fieldset>
        	<legend><?php comment_form_title('Leave a Comment','Leave a Reply to %s'); ?></legend>

        	<?php if ( is_user_logged_in() ) : ?>

			Logged in as <a href="<?php echo get_option('siteurl'); ?>/wp-admin/profile.php"><?php echo $user_identity; ?></a>. <a href="<?php echo get_option('siteurl'); ?>/wp-login.php?action=logout" title="Log out of this account">Logout &raquo;</a>

			<label>Comment: <textarea name="comment" id="comment" cols="50" rows="20"></textarea></label>

		<?php endif; ?>

        	<?php if ( !is_user_logged_in() ) : ?>
			<label>Name: <em>Required</em> <input type="text" name="author" id="author" value="<?php echo $comment_author; ?>" /></label>

			<label>Email: <em>Required, not published</em> <input type="text" name="email" id="email" value="<?php echo $comment_author_email; ?>"/></label>

			<label>Homepage: <input type="text" name="url" id="url" value="<?php echo $comment_author_url; ?>" /></label>

			<label>Comment: <textarea name="comment" id="comment" cols="50" rows="20"></textarea></label>

        	<?php endif; ?>

        	<input type="submit" value="Post Comment" /> <input type="hidden" name="comment_post_ID" value="<?php echo $id; ?>" />

        	</fieldset>

        	<?php do_action('comment_form', $post->ID); ?>

        	</form>

	<?php endif;?>

<?php endif; // if you delete this the sky will fall on your head ?>
{% endcodeblock %}




The only thing you might not understand is the final function:


{% codeblock lang:php %}
<?php do_action('comment_form', $post->ID); ?>
{% endcodeblock %}



which pretty much tells the form what to do with the information. And so there we have it the most simple comments implementaation in the wordpress framework. One last bit to go over is the nice search functionality wordpress has. This is implemented using a theme's search.php file. I'm not going to go too much in depth into this as it is very similar to index.php but I thought I would show you the code just to round off this article, so here you go:



{% codeblock lang:php %}
<?php get_header(); ?>

<form method="get" id="searchform" action="<?php bloginfo('url'); ?>/">
<input type="text" value="<?php the_search_query(); ?>" name="s" id="s" />
<input type="submit" id="searchsubmit" value="Search" />
</form>

<h2>Search Results</h2>

<?php if (have_posts()) : ?>

	<?php while (have_posts()) : the_post(); ?>
		<h3><a href="<?php the_permalink() ?>" rel="bookmark"><?php the_title(); ?></a></h3>
        	<?php the_excerpt('Read the rest of this entry &raquo;'); ?>
        	<?php
        	$year = get_the_time('Y');
        	$month = get_the_time('m');
        	$day = get_the_time('d');
        	?>

        	<a href="<?php echo get_day_link("$year", "$month", "$day"); ?>"><?php the_time('F j, Y'); ?></a>
        	Filed Under: <?php the_category(', '); ?>
        	<?php the_tags('Tags:', ' : ', ''); ?>
	<?php endwhile; ?>
	<?php next_posts_link('&laquo; Older') ?>
	<?php previous_posts_link('Newer &raquo;') ?>

<?php else : ?>
	Not Found
	<p>Sorry, but you are looking for something that isn't here.</p>

<?php endif; ?>

<?php get_sidebar(); ?>
<?php get_footer(); ?>
{% endcodeblock %}

I hope this will help when designing your new wordpress blog, happy coding!
