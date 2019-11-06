---
layout: post
title: How I use Markdown with WordPress
date: 2011-12-30 10:57:23.000000000 -06:00
categories:
- PHP
- Rack App
- Tutorials
- WordPress
tags:
- curl
- heroku
- markdown
- php
- rack
- redcarpet
- sinatra
- wordpress
meta:
  dsq_thread_id: '179 http://baylorrae.com/?p=179'
redirect_from: /blog/2011/12/30/how-i-use-markdown-with-wordpress/
---

I love writing stuff in [Markdown][markdown]. But when I decided to write my blog posts in
MD I wanted to have the same [syntax as GitHub][gfm]. In particular fenced code blocks.

There are a lot of plugins for WordPress to use MD, but most of them were too
complicated for my liking. And they didn't support GitHub's syntax. So I ended
up writing a small [Rack app][rack] with [Sinatra][sinatra] that uses
[Redcarpet][redcarpet] to parse my blog posts.

### The rundown

1. When I save a post, WP sends a POST request to my Heroku app.
2. Heroku parses the data and returns it to WP
3. WP saves the parsed MD into `post_content` and the raw markdown into
`post_content_filtered`
4. When I edit a post, WP grabs the content from `post_content_filtered` instead
of `post_content`

### The Rack app

I've uploaded my [Rack app to GitHub][markdown_rack] so you can see the whole thing, but here's
the important stuff.

```ruby
require 'rubygems'
require "sinatra"
require "redcarpet"

get "/" do
  "I'm alive, I'm alive, I'm ... why are you not excited?"
end

post "/parse" do
  markdown = Redcarpet::Markdown.new(Redcarpet::Render::HTML,
    :autolink => true, :space_after_headers => true, :fenced_code_blocks => true, :no_intra_emphasis => true)
    
  markdown.render(params[:markdown])
end
```

As you can see the code is dead simple. It just grabs the content passed in
`$_POST['markdown']` (from a PHP perspective) and returns the rendered Markdown.

### Modifying WordPress

```php
<?php
function markdown_filter($data, $post) {
  // change this to match your heroku app url
  $ch = curl_init('http://radiant-stone-4578.heroku.com/parse');
  curl_setopt_array($ch, array(
    // don't print the result when done
    CURLOPT_RETURNTRANSFER => true,
    // use a post request and pass on the content
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => 'markdown=' . urlencode($data['post_content'])
  ));
  // grab the parsed content
  $parsed = curl_exec($ch);
  curl_close($ch);
  // back up the original markdown
  $data['post_content_filtered'] = $data['post_content'];
  // save the new parsed html
  $data['post_content'] = $parsed;
  return $data;
}
function get_markdown($content, $id) {
  $post = get_post($id);
  // make sure the post has content saved in post_content_filtered 
  if( $post && !empty($post->post_content_filtered) )
    $content = $post->post_content_filtered;
  return $content;
}
/**
 * I'm not entirely sure why this is done
 * it was used in the "Markdown on Save" plugin
 * and I kept it in case it was needed
 * 
 * @author Mark Jaquith
 */
function get_markdown_filtered($content, $id) {
  return $content;
}
// on post save/update
add_filter( 'wp_insert_post_data' , 'markdown_filter' , '99', 2 );
// when editing a post
add_filter('edit_post_content', 'get_markdown', '99', 2);
// when editing post_content_filtered
add_filter('edit_post_content_filtered', 'get_markdown_filtered', '99', 2);
```

All we do is override the functionality of WP to parse the post content before
it is saved and grab the MD when we edit a post.

The code probably needs to be put into a plugin so it works with all themes, but
I put it into my functions.php file. It's also important to note that the PHP is
based on Mark Jaquith's [Markdown on Save][markdown_on_save].

[markdown]: http://daringfireball.net/projects/markdown/
[gfm]: http://github.github.com/github-flavored-markdown/
[rack]: http://en.wikipedia.org/wiki/Rack_(web_server_interface)
[sinatra]: http://www.sinatrarb.com/
[redcarpet]: https://github.com/tanoku/redcarpet
[markdown_rack]: https://github.com/BaylorRae/Markdown-Parser
[markdown_on_save]: http://txfx.net/wordpress-plugins/markdown-on-save/
