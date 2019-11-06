---
layout: post
title: PHP+Capistrano+Shared Host = Easy Deployments
date: 2012-09-25 23:45:38.000000000 -05:00
categories:
- Capistrano
- Deployments
- PHP
- Shared Hosting
tags:
- capistrano
- deployments
- php
- shared hosting
- tutorial
meta:
  dsq_thread_id: '450 http://baylorrae.com/?p=450'
redirect_from: /blog/2012/09/25/easy-php-deployments-with-capistrano-with-shared-hosting/
---

I wrote a Rails app that was deployed with Capistrano and I loved it. Every time
I made a change, one file or a dozen, I could get all the files up in a single
command. In addition, if anything was broken I could revert to the previous
deploy with `cap deploy:rollback`.

While using Capistrano is considered a "standard" with Rails, it's something
that I think should be used more with PHP as well. When I built my last website
with PHP I reverted back to my FTP client deploys, and it was a nightmare. I
worried about the order in which I uploaded each file and making sure I uploaded
all of my changes.

I'm aware of the services like [Beanstalk][beanstalk] that let you do a `git push` and will
move the modified files for you. But, I'm not a big fan of sharing my entire
repository or giving access to my server. Plus, I'm a little cheap, especially
when I know I can write my own system for free. It may not be as fancy and it
may require a little more work, but I like having complete control.

### The Requirement

I don't want to explain how to make deployments with Capistrano. There are a lot
of tutorials on the web that do a much better job than I can. So, if you're not
familiar with Capistrano, you will need to follow a tutorial. The [Capistrano
Wiki][capistrano_wiki] has quite a few resources that you may find helpful.

However, there is one modification that is far different than most Capistrano
tutorials. On my shared host, I wasn't allowed to change the `DocumentRoot` for my
primary domain. As a result, I had to create a symlink from `~/public_html` to the
latest release on every deploy. While it sounds difficult, it's actually very
easy.

### The Deployment Strategy

This is where all the magic of Capistrano takes place. It's where you get to
decide what takes place and when. You can define your entire deploy strategy
here and Capistrano will make sure it happens, every time.

```ruby
set :user, "USERNAME"

# about the application
set :application, "example.com"
set :port, 2222

# the type and location of the repository
set :scm, :git
set :repository, "."

# how and where it should be uploaded
set :deploy_to, "/home/#{user}/sites/#{application}"
set :deploy_via, :copy

# what should be left behind in the copy
set :copy_exclude, %w[.git .DS_Store .gitignore .gitmodules .sass-cache sass Capfile config config.rb]

# which server to use
server application, :app

# shared hosting != sudo privileges
set :use_sudo, false

# only keep the last four releases on the server
set :keep_releases, 4

namespace :deploy do
  desc "Add symlink to `public_html`"
  task :public_html_symlink, :rols => :app do
    on_rollback do
      if previous_release
        run "rm -f ~/public_html; ln -s #{previous_release} ~/public_html; true"
      else
        logger.important "no previous release to rollback to, rollback of symlink skipped for ~/public_html"
      end
    end
    
    run "rm -f ~/public_html && ln -s #{release_path} ~/public_html"
  end
  
  desc "Remove group write privileges"
  task :finalize_update do
    transaction do
      run "chmod -R g-w #{release_path}"
      
       # copy all items from the shared folder
      shared_children.map do |d|
        if d.rindex('/')
          run "rm -rf #{latest_release}/#{d} && mkdir -p #{latest_release}/#{d.slice(0..(d.rindex('/')))}"
        else
          run "rm -rf #{latest_release}/#{d}"
        end
        
        run "ln -s #{shared_path}/#{d.split('/').last} #{latest_release}/#{d}"
      end
    end
  end
  
  # disable rails stuff
  task :migrate do; end
  task :restart do; end
end

# removes older releases
after "deploy:update", "deploy:cleanup"

after "deploy:create_symlink", "deploy:public_html_symlink"
```

The most important change to the default deployment is on line 38. It deletes
the default `~/public_html` directory and replaces it with a symlink to the latest
release. A very simple command that makes the whole thing work.

The result is when I want to push the changes I made I can run `cap deploy` and my
entire application is uploaded. I don't have to worry about individual files,
and if something breaks I can always return to the previous release with `cap
deploy:rollback`.

[beanstalk]: http://beanstalkapp.com/
[capistrano_wiki]: https://github.com/capistrano/capistrano/wiki
